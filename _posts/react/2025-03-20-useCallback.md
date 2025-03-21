---
layout: post
title: REACT - useCallback 과 메모리 사용
category: REACT
---

# React의 useCallback 이해하기 🔍

React의 **`useCallback`**은 **메모이제이션(memoization)**을 통해 **함수가 불필요하게 재생성되는 것을 방지**하는 Hook입니다.

---

## 📌 왜 쓰는가? (문제 상황)

React 컴포넌트 내부의 함수는 컴포넌트가 리렌더링될 때마다 새로 생성됩니다. 이는 불필요한 렌더링이나 성능 저하를 초래할 수 있습니다.

```jsx
const MyComponent = () => {
  const handlePress = () => {
    console.log('Pressed!');
  };

  return <Button onPress={handlePress} />;
};
```

위 코드처럼 작성하면 컴포넌트가 리렌더링될 때마다 `handlePress`가 매번 재생성됩니다.

---

## ✅ 해결 방법 (`useCallback`)

```jsx
const MyComponent = () => {
  const handlePress = useCallback(() => {
    console.log('Pressed!');
  }, []);

  return <Button onPress={handlePress} />;
};
```

- `useCallback`으로 감싸면 의존성 배열(`[]`)의 값이 변하지 않는 한 동일한 함수가 재사용됩니다.
- 빈 배열(`[]`)을 사용하면 최초 마운트 시 한 번만 생성되고 계속 재사용됩니다.

---

## 📌 useCallback의 캐싱 유지와 해제 시점

### 컴포넌트 마운트 시
- 최초 컴포넌트 마운트 시 함수가 메모리에 저장됩니다.

### 컴포넌트 업데이트 시
- 의존성 배열 값이 변경되지 않으면 기존 함수가 유지됩니다.
- 의존성이 변경되면 새로운 함수가 생성됩니다.

### 컴포넌트 언마운트 시
- 컴포넌트가 화면에서 사라지면 React가 자동으로 메모리를 정리하여 캐싱된 함수가 제거됩니다.

**즉, 다른 페이지로 이동하거나 컴포넌트가 사라지면 캐싱된 함수도 즉시 제거됩니다.**

---

## 📌 메모리 관리 흐름 정리

| 상황 | 행동 | 메모리 상태 |
|---|---|---|
| 최초 마운트 시 | 함수가 최초로 생성되어 저장됨 | ✅ 메모리 유지 |
| 리렌더링 시 (의존성 변화 없음) | 동일 함수 재사용 | ✅ 메모리 유지 |
| 리렌더링 시 의존성 변경 | 새로운 함수가 생성되고 이전 함수 폐기 | 🔄 메모리 업데이트 |
| 언마운트 또는 페이지 이동 시 | React가 자동으로 메모리에서 제거 | ❌ 메모리 제거 |

---

## 📌 불필요한 캐싱이 유지될까?

- **아닙니다.** 컴포넌트가 언마운트되면 메모리에서 즉시 해제됩니다.
- 앱 전체에서 유지되는 전역적인 캐싱이 아닙니므로 안심해도 됩니다.

---

## 📌 언제 사용하면 좋은가?

- 성능 최적화가 필요할 때
- 자주 리렌더링되는 리스트 컴포넌트의 렌더링 함수 (`FlatList`의 `renderItem` 등)
- 자식 컴포넌트에 props로 넘겨지는 함수가 빈번히 변경될 때

---

## 🚩 주의 사항

- 모든 함수에 무조건적으로 적용할 필요는 없습니다.
- 성능 최적화가 명확히 필요한 경우에만 사용하세요.

