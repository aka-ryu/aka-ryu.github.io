---
layout: post
title: TECH - React Compiler RC
category: TECH
---

## React Compiler RC란?

React Compiler RC(Release Candidate)는 React 팀이 발표한 **자동 렌더링 최적화 컴파일러**입니다.

기존에는 개발자가 `React.memo`, `useMemo`, `useCallback` 같은 최적화 코드들을 직접 작성해야 했지만,
React Compiler는 **이런 최적화를 컴파일 타임에 자동으로 처리해주는 역할**을 합니다.

한마디로, 개발자가 신경 쓰지 않아도 React가 **자동으로 메모이제이션과 리렌더링을 관리해 주는 기술**입니다.

---

## 특징

### 1. 자동 메모이제이션
React Compiler는 **상태 변화에 따라 가져와야 하는 부분만 리렌더링 하도록 캡시해주며**
그 가지를 자동적으로 관리합니다.

### 2. 컴파일 타임 최적화
React가 런타임에 메모이제이션을 결정했다면, React Compiler는 **빌드 시점에 무시해주는 역할**을 합니다. 정적 분석 기능을 통해 자동적으로 처리합니다.

### 3. 넓은 빌드 툴 지원
- SWC (Next.js 15.3.1+)
- Babel (React 17, 18, 19 모두 지원)
- Vite, Metro 등 다양한 빌드 환경에서 사용 가능

### 4. React 19부터 공식 지원
React 19에서는 컴파일러 기능이 기본 포함되어 있고,
React 17, 18에서도 플랫귀인 설치 후 사용 가능합니다.

---

## React Compiler 없이 기존적으로 작성하는 방식

이전의 React에서는 **리렌더링 최적화**를 위해 다음과 같은 코드를 필요변수 배정했어야 했습니다.

```tsx
import { useState, useCallback } from 'react';

function Button({ onClick, label }: { onClick: () => void; label: string }) {
  console.log('Button rendered');
  return <button onClick={onClick}>{label}</button>;
}

export default function App() {
  const [count, setCount] = useState(0);
  const [other, setOther] = useState(0);

  const increment = useCallback(() => setCount((c) => c + 1), []);
  const toggle = () => setOther((o) => o + 1);

  return (
    <div>
      <Button onClick={increment} label={`Count: ${count}`} />
      <button onClick={toggle}>Toggle</button>
    </div>
  );
}
```

- `increment` 함수를 `useCallback`으로 무시해야 했고,
- `Button` 컨테이너는 복잡하면 `React.memo`로 감사해야 했어요.

---

## React Compiler 적용 시

React Compiler를 사용하면 **위 코드에서 useCallback, React.memo를 아직 쓰지 않아도 됩니다.**

```tsx
import { useState } from 'react';

function Button({ onClick, label }: { onClick: () => void; label: string }) {
  console.log('Button rendered');
  return <button onClick={onClick}>{label}</button>;
}

export default function App() {
  const [count, setCount] = useState(0);
  const [other, setOther] = useState(0);

  const increment = () => setCount((c) => c + 1);
  const toggle = () => setOther((o) => o + 1);

  return (
    <div>
      <Button onClick={increment} label={`Count: ${count}`} />
      <button onClick={toggle}>Toggle</button>
    </div>
  );
}
```

### ✅ React Compiler가 아래와 같이 자동 판단
- `increment`가 다시 생성되어도 Button이 리렌더링되지 않도록 자동 최적화
- `toggle`를 눌러도 Button은 리렌더링하지 쥜

> React Compiler가 의존성, 함수, 상태 변화를 자동 통계해서 **최소한 리렌더링만 일어나도록 컴파일 시점에 커뮤니쯀화**합니다.

---

## 잠시한 정보

| 특성 | 결론 |
|---------|---------|
| 개발자가 메모이제이션 개선 가능 | 복잡한 자동 최적화 가능 |
| 코드 가능성 감을 | 컨테이너 복잡도 감소 |
| 빌드 시간 초과 가능 | 컨테이너 지배와 마침 가능 |
| 공중 통계 사용 필요 | 빌드 환경에 따라 발생 가능 |

---

## 정리

- React Compiler RC는 **리렌더링 최적화의 게임 찌이어**다.
- React 19부터는 컴파일러 기능이 기본 포함되며, React.memo, useCallback이 필요없는 React 코드가 가능해진다.
- 개발 속도가 빠른 것과, 코드 혹은 UI 시스템의 가능성이 더 좋아지는 것은 가해적이다.
- 다른 공적적인 테스팅이 필요하며, RC 버전인만\ud큼 결과 검사는 필요하다.
