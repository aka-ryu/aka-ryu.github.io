---
layout: post
title: ARCHITECTURE - APP IN APP
category: ARCHITECTURE
---


## JWT 토큰 무효화: 블랙리스트 + Redis TTL 전략

JWT는 기본적으로 stateless라서 한 번 발급하면 서버에서 강제로 "폐기"하는 게 어렵다. 그래서 로그아웃이나 강제 로그아웃 같은 상황에서는 **블랙리스트 전략**을 써야 한다.

내가 쓴 방식은 이렇다:

---

### ✅ 핵심 아이디어

* JWT를 **디코딩**해서 `exp` 만료 시간 뽑아낸다
* `exp - 현재 시간`만큼 Redis에 **TTL 설정해서 저장**한다
* 그 TTL 동안은 Redis에서 검사해서 **요청을 거부**한다
* 시간이 지나면 Redis에서 **자동 삭제됨** (깔끔함)

---

### 🔧 예시 코드

#### 1. 로그아웃 시 처리

```ts
import jwt from 'jsonwebtoken';
import { createHash } from 'crypto';

function hashToken(token: string) {
  return createHash('sha256').update(token).digest('hex');
}

const decoded = jwt.decode(token);
const exp = decoded?.exp;

if (exp) {
  const ttl = exp - Math.floor(Date.now() / 1000);
  if (ttl > 0) {
    const tokenHash = hashToken(token);
    await redis.set(`bl:${tokenHash}`, 'blacklisted', 'EX', ttl);
  }
}
```

#### 2. 요청 시 검사 (Auth Guard 등)

```ts
const tokenHash = hashToken(token);
const isBlacklisted = await redis.get(`bl:${tokenHash}`);

if (isBlacklisted) {
  throw new UnauthorizedException('Token is blacklisted');
}
```

---

### ✅ 이 방식의 장점

* **즉시 무효화 가능** → 로그아웃하면 바로 차단됨
* **Redis TTL로 자동 정리** → 토큰 만료되면 블랙리스트도 삭제됨
* **빠름** → Redis는 in-memory니까 성능도 좋음

---

### ⚠️ 주의할 점

* `jwt.decode()`는 서명 검증 안 한다 → **위조 방지 위해선 `verify()`도 같이 써야 됨**
* Redis에 저장할 땐 **토큰 해시로 키 만드는 게 안전** (직접 저장은 비추)

---

### ✅ 결론

* JWT 무효화가 필요하다면, 블랙리스트 + TTL 방식이 꽤 현실적이고 깔끔함
* Redis TTL 덕분에 관리 부담도 없음
* 실제 프로젝트에서 무리 없이 적용 가능

