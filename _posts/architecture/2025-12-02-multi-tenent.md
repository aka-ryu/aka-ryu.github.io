---
layout: post
title: ARCHITECTURE - 멀티테넌트 동적 SSL 발급 (DB + DNS + Caddy)
category: ARCHITECTURE
---


멀티테넌트 SaaS를 운영하다 보면, 고객사마다 **커스텀 도메인**을 제공해야 하는 상황이 온다. 단순히 서브도메인만 주는 게 아니라, `customer.com` 같은 **완전한 독립 도메인**을 연결해줘야 할 때가 있다.

이걸 구현하려면 **DB에서 도메인 관리 → DNS 검증 → SSL 자동 발급**까지 한 흐름으로 처리해야 한다. 내가 구축한 방식은 이렇다:

---

## 1. 전체 구조

```
[고객 도메인 등록]
       ↓
[DB에 도메인 저장] → Company 테이블에 domain 컬럼
       ↓
[고객이 DNS 설정] → CNAME 또는 A 레코드로 우리 서버 지정
       ↓
[Caddy on-demand TLS] → 요청 시 /api/verify-domain으로 검증
       ↓
[SSL 자동 발급] → Let's Encrypt에서 인증서 발급
       ↓
[서비스 연결] → reverse_proxy로 Next.js 앱 연결
```

---

## 2. DB 설계 (도메인 관리)

회사(Company) 테이블에 도메인 정보를 저장한다:

```prisma
model Company {
  id          String   @id @default(cuid())
  name        String
  domain      String?  @unique  // 커스텀 도메인 (예: customer.com)
  tier        String   @default("SHARED")  // SHARED | PREMIUM
  // ... 기타 필드
}
```

### 티어별 도메인 정책
- **SHARED**: 공용 도메인 사용 (예: `app.ourservice.com`)
- **PREMIUM**: 커스텀 도메인 허용 (예: `customer.com`)

---

## 3. DNS 설정 안내

고객이 자기 도메인을 연결하려면 DNS 레코드를 설정해야 한다:

### CNAME 방식 (권장)
```
customer.com → CNAME → app.ourservice.com
```

### A 레코드 방식
```
customer.com → A → 12.34.56.78 (EC2 IP)
```

고객에게 안내할 때는 **CNAME 방식을 권장**한다. IP가 바뀌어도 대응 가능하니까.

---

## 4. Caddy 설정 (on-demand TLS)

Caddy는 **on-demand TLS**라는 기능이 있다. 요청이 들어올 때 **동적으로 SSL 인증서를 발급**해준다.

### Caddyfile

```caddyfile
# 모든 도메인에 대해 자동 SSL 발급
{
    on_demand_tls {
        ask http://localhost:3000/api/verify-domain
    }
}

# 모든 HTTPS 요청 처리
:443 {
    tls {
        on_demand
    }
    reverse_proxy localhost:3000
}

# HTTP → HTTPS 리다이렉트
:80 {
    redir https://{host}{uri} permanent
}
```

### 핵심 포인트
- `ask` 옵션으로 **우리 API에 도메인 검증 요청**을 보냄
- API가 `200 OK` 응답하면 SSL 발급 진행
- API가 `4xx/5xx` 응답하면 SSL 발급 거부

---

## 5. 도메인 검증 API

Caddy가 SSL 발급 전에 호출하는 API다. DB에 등록된 도메인인지 확인한다:

```ts
// /api/verify-domain/route.ts
import { NextRequest, NextResponse } from "next/server";
import prisma from "@/lib/prisma";

export async function GET(request: NextRequest) {
  const domain = request.nextUrl.searchParams.get("domain");

  if (!domain) {
    return NextResponse.json({ error: "Domain required" }, { status: 400 });
  }

  try {
    // DB에서 해당 도메인을 가진 회사 조회
    const company = await prisma.company.findFirst({
      where: {
        domain: domain,
        deletedAt: null,
      },
    });

    if (company) {
      // 등록된 도메인 → SSL 발급 허용
      return NextResponse.json({ allowed: true }, { status: 200 });
    } else {
      // 미등록 도메인 → SSL 발급 거부
      return NextResponse.json({ allowed: false }, { status: 404 });
    }
  } catch (error) {
    console.error("[verify-domain] Error:", error);
    return NextResponse.json({ error: "Internal error" }, { status: 500 });
  }
}
```

---

## 6. SSL 인증서 저장 위치 (Caddy)

Caddy는 발급받은 인증서를 로컬에 캐싱한다:

```bash
# 인증서 저장 경로
/var/lib/caddy/.local/share/caddy/certificates/acme-v02.api.letsencrypt.org-directory/

# 구조
├── customer1.com/
│   ├── customer1.com.crt
│   └── customer1.com.key
├── customer2.com/
│   ├── customer2.com.crt
│   └── customer2.com.key
```

### 인증서 정리 (도메인 삭제 시)

도메인을 더 이상 사용하지 않으면 인증서 찌꺼기가 남는다. 정리하려면:

```bash
# 특정 도메인 인증서 삭제
sudo rm -rf /var/lib/caddy/.local/share/caddy/certificates/acme-v02.api.letsencrypt.org-directory/customer.com/

# OCSP 캐시도 정리
sudo rm -rf /var/lib/caddy/.local/share/caddy/ocsp/

# Caddy 재시작
sudo systemctl restart caddy
```

---

## 7. 미들웨어에서 도메인 기반 라우팅

요청이 들어오면 **어떤 회사의 요청인지** 도메인으로 판단한다:

```ts
// middleware.ts
import { NextRequest, NextResponse } from "next/server";

export async function middleware(request: NextRequest) {
  const hostname = request.headers.get("host") || "";
  
  // 도메인으로 회사 식별
  const company = await getCompanyByDomain(hostname);
  
  if (!company) {
    return NextResponse.redirect(new URL("/404", request.url));
  }
  
  // 요청에 회사 정보 주입
  const response = NextResponse.next();
  response.headers.set("x-company-id", company.id);
  
  return response;
}
```

---

## 8. 전체 플로우 정리

| 단계 | 작업 | 담당 |
|------|------|------|
| 1 | 마스터 페이지에서 회사 + 도메인 등록 | Admin |
| 2 | DB에 도메인 저장 | Server |
| 3 | 고객에게 DNS 설정 안내 | Admin |
| 4 | 고객이 CNAME/A 레코드 설정 | Customer |
| 5 | 첫 요청 시 Caddy가 verify-domain API 호출 | Caddy |
| 6 | API가 DB 확인 후 200 응답 | Server |
| 7 | Caddy가 Let's Encrypt에서 SSL 발급 | Caddy |
| 8 | 이후 요청은 SSL로 정상 처리 | Caddy |

---

## 9. 운영 시 주의사항

### Let's Encrypt 제한
- **주당 50개** 인증서 발급 제한
- 너무 자주 삭제/재발급하면 제한 걸릴 수 있음

### DNS 전파 시간
- DNS 변경 후 **최대 48시간** 걸릴 수 있음
- 보통은 몇 분 ~ 몇 시간 내 적용됨

### 도메인 삭제 시
- DB에서 도메인 제거하면 다음 요청부터 SSL 발급 거부됨
- 기존 인증서는 만료될 때까지 캐시에 남음 (수동 삭제 가능)

---

## 10. 마무리

멀티테넌트에서 커스텀 도메인 + 자동 SSL은 생각보다 구현이 까다롭다. 하지만 **Caddy의 on-demand TLS**를 활용하면 꽤 깔끔하게 처리할 수 있다.

핵심은 이거다:
- **DB가 도메인의 진실의 원천** (Source of Truth)
- **Caddy는 DB한테 물어보고 SSL 발급 여부 결정**
- **인증서 관리는 Caddy가 알아서 함**

이 구조면 고객사가 100개든 1000개든, 각자 도메인으로 서비스 제공이 가능하다.
