---
layout: post
title: NEXTJS - Server Only
categories: NEXTJS
---
# Next.js 에서 `"use client"`, `server-only`, `client-only` 개념 정리

Next.js에서는 클라이언트 컴포넌트와 서버 컴포넌트를 명확히 구분하는데,
이걸 명시적으로 알려주는 키워드들이 있다.

대표적으로 `'use client'`, `server-only`, `client-only` 가 있는데,
그 의미와 역할을 정리해봤다.

---

## ✅ 'use client' 는 무엇인가?

> "이 파일은 클라이언트에서만 실행해야 한다"고 Next.js에게 알려주는 지시어이다.

기본적으로 Next.js는 모든 컴포넌트를 **서버 컴포넌트(Server Component)** 로 처리하려고 한다.

하지만 브라우저에서만 실행되는 기능들 (`useState`, `useEffect`, `onClick` 등)을 쓰는 경우엔
이 컴포넌트가 클라이언트 전용이라는 걸 명시적으로 알려줘야 한다.

그게 바로 `'use client'` 이다.

```tsx
'use client';

import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```


## ✅ `server-only` 는 무엇인가?

> 이 파일은 오직 서버에서만 사용되어야 한다고 명시하는 선언이다.

### 사용 예시:

```ts
// lib/db.ts
import 'server-only';

export async function getUser() {
  return await db.query('SELECT * FROM users');
}
```

이렇게 작성하면 이 모듈이 클라이언트 코드에서 실수로 import 되었을 때, Next.js가 경고를 해주거나 빌드 에러를 발생시켜준다.

**DB 접근, 인증 토큰 로직, 민감한 정보 처리** 등은 반드시 서버에서만 실행되어야 하므로
이런 경우 `server-only`를 사용하는 것이 안전하다.

---

## ✅ `client-only` 는 무엇인가?

> 서버 컴포넌트 환경에서도 무조건 클라이언트에서만 실행되어야 함을 명시하는 것이다.

다만 일반적으로는 `client-only`라고 따로 import 하기보다는,
그냥 `'use client'` 지시어를 사용하여 클라이언트 컴포넌트임을 명시하는 방식으로 쓰는 것이 일반적이다.

```tsx
'use client';

import { useState } from 'react';

export default function Modal() {
  const [open, setOpen] = useState(false);
  return <div>{open && <div>모달 내용</div>}</div>;
}
```

---

## 🧠 요약

| 키워드            | 의미                       | 사용 위치       | 목적            |
| -------------- | ------------------------ | ----------- | ------------- |
| `'use client'` | 클라이언트 전용 컴포넌트            | 컴포넌트 파일 최상단 | CSR 렌더링 지정    |
| `server-only`  | 서버 전용 모듈                 | 민감한 로직 파일   | 보안/안전 보호      |
| `client-only`  | 클라 전용 (사실상 'use client') | 거의 안 쓰임     | 클라에서만 실행됨을 강조 |

---

Next.js는 App Router 구조에서 기본이 서버 컴포넌트이기 때문에,
이런 명시적인 구분을 잘 해주는 게 성능이나 보안 측면에서도 중요한 부분이다.
