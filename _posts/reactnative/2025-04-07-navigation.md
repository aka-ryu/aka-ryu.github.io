---
layout: post
title: REACTNATIVE - navigation props? useNavigation hooks? 선택
category: REACTNATIVE
---



## props 구조 분해 (Destructuring)

기본적으로 컴포넌트가 props 를 받을 때 더 깔끔하게 사용할 수 있는 방식이다.

### 기본
```tsx
const MyComponent = (props) => {
  return <Text>{props.title}</Text>;
};
```

### 구조 분해 할당
```tsx
const MyComponent = ({ title }) => {
  return <Text>{title}</Text>;
};
```

조금 더 보기 쉽고 관리하기 편하다.

---

## props drilling 이란?

부모 컴포넌트에서 받은 props 를 계속해서 자식, 손자, 증손자에게 전달하는 것.
규모가 커질수록 복잡도가 올라간다.

```tsx
const Parent = () => {
  const user = { name: '재현' };
  return <Child user={user} />;
};

const Child = ({ user }) => {
  return <GrandChild user={user} />;
};

const GrandChild = ({ user }) => {
  return <Text>{user.name}</Text>;
};
```

이렇게 깊어지면 관리가 힘들어지기 때문에 Context API 나 Zustand 같은 전역 상태 관리 라이브러리로 해결하는 게 좋다.

---

## Stack / Tab / Modal 네비게이션 차이

| 구분 | Stack Navigator | Tab Navigator | Modal Navigator |
|---|---|---|---|
| 역할 | 화면을 쌓아가며 이동 | 탭 간 평면 이동 | 임시로 화면을 띄움 |
| 목적 | 단계별 화면 이동 (예: 상품 상세 → 결제 → 완료) | 주요 메뉴 이동 | 알림, 약관, 확인 등 임시 화면 |
| UI | 기본 좌우 스와이프 (iOS) | 하단 탭 UI | 배경을 덮고 나타나는 화면 |
| 사용 예시 | 결제 플로우 | 홈/스토어/마이페이지 | 약관 동의 모달 등 |

대부분은 Stack 안에 Tab, Tab 안에 Stack 형태로 조합해서 사용하게 된다.

---

## props vs hook 으로 navigation 사용하기

| 구분 | props.navigation | useNavigation() (hook) |
|---|---|---|
| 특징 | 스크린 컴포넌트에 자동 전달 | 컴포넌트 어디서든 사용 가능 |
| 사용 위치 | 네비게이터에 등록된 스크린 컴포넌트 | 깊은 자식 컴포넌트 |
| 장점 | 직관적, 빠르다 | props drilling 을 방지할 수 있다 |
| 단점 | 자식 컴포넌트로 계속 props 전달해야 한다 | React 컴포넌트 함수 안에서만 사용 가능 |

### 사용 예

스크린 컴포넌트에서는 props 로 navigation 을 사용
```tsx
const ScreenA = ({ navigation }) => {
  navigation.navigate('ScreenB');
};
```

자식의 자식 컴포넌트에서는 hook 사용
```tsx
import { useNavigation } from '@react-navigation/native';

const SomeDeepChildComponent = () => {
  const navigation = useNavigation();
  navigation.navigate('ScreenB');
};
```

결론적으로,
- **Stack / Tab 에 등록된 스크린**은 `props.navigation`
- **일반 자식 컴포넌트**는 `useNavigation()` 을 사용하는 게 깔끔하다.

> 참고: route 도 동일하게 사용할 수 있다.
```tsx
import { useRoute } from '@react-navigation/native';
const route = useRoute();
console.log(route.params);
```

---

## 요약

- **props destructuring**: 구조 분해로 props 깔끔하게 사용
- **props drilling**: 깊은 컴포넌트 전달 시 복잡 → 상태 관리 필요
- **Stack / Tab / Modal 차이**: 단계적 / 평면 / 임시 화면
- **navigation 사용법**:
  - 스크린 컴포넌트: `props.navigation`
  - 자식 컴포넌트: `useNavigation()`

---



