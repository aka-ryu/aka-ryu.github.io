---
layout: post
title: TYPESCRIPT - TypeScript에서 DTO 정의 Class, Interface, Type의 차이와 선택 가이드
category: TYPESCRIPT
---

1. **Class 사용**  
Class는 다음과 같은 이유로 DTO(Data Transfer Object) 정의에 적합합니다:

- **유효성 검사 및 직렬화 지원**: 데코레이터(`@IsString`, `@IsOptional` 등)를 활용한 유효성 검사 및 변환이 가능.
- **NestJS 통합**: `class-validator`, `class-transformer`와 같은 라이브러리와 함께 사용 가능.
- **Swagger 문서 통합**: NestJS 데코레이터(`@ApiProperty`)를 사용하여 Swagger 문서를 자동 생성할 수 있음.

### 예제
```typescript
import { IsOptional, IsString, IsNumber } from 'class-validator';
import { ApiProperty } from '@nestjs/swagger';

export class CreateUserRequestDTO {
  @ApiProperty({ description: '사용자 이름', example: 'John Doe' })
  @IsString()
  name: string;

  @ApiProperty({ description: '사용자 이메일', example: 'john.doe@example.com' })
  @IsString()
  email: string;

  @ApiProperty({ description: '사용자 나이', example: 30 })
  @IsOptional()
  @IsNumber()
  age?: number;
}
```

### 장점
- **유효성 검사 가능**: 입력 값 검증이 필요한 경우 매우 적합.
- **Swagger 문서 통합**: NestJS와 함께 Swagger 문서 자동 생성.
- **유연한 직렬화**: 데이터 변환 및 직렬화가 필요한 경우 유용.

### 단점
- 조금 더 복잡한 구조로 인해 간단한 데이터 정의에는 다소 과함.

---
<br><br><br>
2. **Interface 사용**  
`interface`는 간단하게 객체의 구조를 정의하는 데 유용합니다.  
주로 유효성 검증이나 변환이 필요하지 않은 경우에 사용됩니다.

### 예제
```typescript
export interface CreateUserRequestDTO {
  name: string;
  email: string;
  age?: number;
}
```

### 장점
- **간결함**: 간단한 데이터 구조 정의에 적합.
- **타입 확장**: 다른 인터페이스와 쉽게 병합 가능.
- **타입스크립트 스타일**: 함수형 코드베이스에서 유용.

### 단점
- **유효성 검사 불가**: 입력 값 검증이 필요하면 추가 구현 필요.
- Swagger 같은 문서화 툴과 바로 연결하기 어렵다.

---
<br><br><br>
3. **Type 사용**  
`type`은 주로 유니온 타입이나 특정 데이터 구조를 확장 없이 단순히 정의할 때 사용됩니다.

### 예제
```typescript
export type CreateUserRequestDTO = {
  name: string;
  email: string;
  age?: number;
};
```

### 장점
- 간단한 데이터 정의에 적합.
- 유니온 타입 등 복잡한 타입 조합에 유리.

### 단점
- **유효성 검사 불가**.
- 클래스 데코레이터나 Swagger 문서 생성과 연동이 어렵다.

---

## 결론
- **유효성 검사 및 문서화가 필요한 경우**: `class`
- **간단한 데이터 정의**: `interface` 또는 `type`