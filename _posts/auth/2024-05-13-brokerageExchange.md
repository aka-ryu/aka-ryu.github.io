---
layout: post
title: AUTH - 애플 소셜로그인 개발환경 설정방법
category: AUTH
---

# 🍎 Apple 로그인 개발 환경 구성 (ngrok 활용)

Apple 로그인은 보안 정책상 리다이렉트 URL에 반드시 `HTTPS`를 사용해야 하며,
`localhost`, `http://`, `127.0.0.1`과 같은 도메인은 **허용되지 않습니다.**

따라서 로컬 개발 환경에서는 [ngrok](https://ngrok.com/)을 이용해 `https` 주소를 임시 발급받아 테스트해야 합니다.

---

## ✅ 전체 아키텍처 흐름

```text
┌────────────────────┐             ┌────────────────────────────┐
│   Developer Mac    │  ▶ Apple ▶ │  https://xxxx.ngrok.io     │
│ (localhost:3000)   │ ◀ Redirect │  (ngrok → localhost tunnel)│
└────────────────────┘             └────────────────────────────┘
```

---

## ✅ ngrok HTTPS 서브도메인 생성 절차

### 1. ngrok 설치 및 계정 등록

```bash
brew install ngrok               # Mac 기준
ngrok config add-authtoken YOUR_AUTH_TOKEN  # 토큰 등록 (1회)
```

### 2. 로컬 서버에 ngrok 연결

```bash
ngrok http 3000
```

예시 결과:
```
Forwarding  https://abcd1234.ngrok.io  ->  http://localhost:3000
```

### 3. Apple Developer Console 설정

- **Service ID** 생성 or 기존 로그인용 Service ID 사용
- **Sign In with Apple > Configure**
  - ✅ Web Domain: `abcd1234.ngrok.io`
  - ✅ Return URL: `https://abcd1234.ngrok.io/auth/login/apple/callback`

---

## ✅ .env 설정 (개발용)

```env
APPLE_CLIENT_ID=com.myapp.dev.login
APPLE_CALLBACK_URL=https://abcd1234.ngrok.io/auth/login/apple/callback
```

NestJS의 passport-apple 설정 시 `callbackURL`에 위 값을 동적으로 할당:

```ts
callbackURL: process.env.APPLE_CALLBACK_URL,
```

---

## ✅ 주의사항

| 항목 | 내용 |
|------|------|
| ✅ `localhost`, `127.0.0.1` 사용 | ❌ Apple에서 허용하지 않음 |
| ✅ HTTP 프로토콜 | ❌ 불가, 반드시 HTTPS |
| ✅ 매번 도메인 변경 | 무료 ngrok의 경우 도메인 고정 불가 (유료 필요) |
| ✅ .env 자동화 | 필요 시 ngrok 주소를 출력 받아 .env에 반영하는 스크립트 작성 가능 |

---

## ✅ 요약

- Apple 로그인은 **HTTPS 도메인만 허용**
- 개발 환경에서는 `ngrok`을 통해 https 서브도메인을 생성
- Apple Console에는 **Web Domain + Return URL** 모두 등록 필수
- NestJS `.env` 파일에서 `APPLE_CALLBACK_URL` 동적으로 분기 처리

---


