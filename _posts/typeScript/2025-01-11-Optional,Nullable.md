---
layout: post
title: TYPESCRIPT - TypeScript에서 Optional 필드와 Nullable 필드의 차이 및 사용 가이드
category: TYPESCRIPT
---

# TypeScript에서 Optional 필드와 Nullable 필드 비교

## 1. 옵션 필드 (`?`)를 사용하는 방식

```typescript
userCode?: string;
```

- 이 방식은 해당 필드가 **존재하지 않을 수도 있음**을 명시합니다.
- 객체를 생성할 때 `userCode`가 아예 포함되지 않을 가능성을 의미합니다.

### 예제
```typescript
const request1: AdminSendFcmRequestDTO = {
  sendType: 'one',
  title: 'Test Title',
  content: 'Test Content',
  imageUrl: 'http://example.com/image.jpg',
};

const request2: AdminSendFcmRequestDTO = {
  sendType: 'one',
  title: 'Test Title',
  content: 'Test Content',
  imageUrl: 'http://example.com/image.jpg',
  userCode: '12345',
};
```

---

## 2. `null` 가능성을 명시하는 방식

```typescript
userCode: string | null;
```

- 이 방식은 필드가 **항상 존재하지만, `null` 값이 될 수도 있음**을 명시합니다.
- `userCode`가 반드시 포함되어야 하지만, 값이 설정되지 않을 수도 있음을 나타냅니다.

### 예제
```typescript
const request1: AdminSendFcmRequestDTO = {
  sendType: 'one',
  title: 'Test Title',
  content: 'Test Content',
  imageUrl: 'http://example.com/image.jpg',
  userCode: null,
};

const request2: AdminSendFcmRequestDTO = {
  sendType: 'one',
  title: 'Test Title',
  content: 'Test Content',
  imageUrl: 'http://example.com/image.jpg',
  userCode: '12345',
};
```

---

## 어떤 방식이 더 일반적인가?

### **옵션 필드 (`?`) 사용이 일반적입니다.**

1. **TypeScript 스타일**:
   - `?`를 사용하는 방식은 TypeScript에서 **필드가 아예 포함되지 않을 가능성**을 나타내며 더 직관적입니다.
   - 불필요한 `null` 체크를 피할 수 있습니다.

2. **NestJS와 같은 프레임워크**:
   - `class-validator`를 사용할 때도 옵션 필드를 처리하기 용이합니다. (`@IsOptional()`과 잘 어울림)

3. **가독성**:
   - 옵션 필드를 사용하는 방식이 코드베이스에서 널 가능성을 간결하게 표현합니다.

---

## 결론 및 추천

- **`userCode` 필드가 있을 수도, 없을 수도 있다면**: 
  - `userCode?: string`을 사용하는 것이 더 일반적이고 직관적입니다.
  
- **`userCode`가 반드시 존재하지만, 값이 설정되지 않을 가능성이 있다면**: 
  - `userCode: string | null`을 사용하세요.

### 실제 예
```typescript
export class AdminSendFcmRequestDTO {
  sendType: 'all' | 'one';
  userCode?: string; // 필드가 없을 수도 있음
  title: string;
  content: string;
  imageUrl: string;
  data?: AdminSendFcmDataDTO;
}
```

