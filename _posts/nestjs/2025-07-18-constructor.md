---
layout: post
title: NESTJS - constructor 역활
category: NESTJS
---
### NestJS에서 constructor의 역할

NestJS에서 constructor는 의존성 주입을 위한 기본 통로다. 단순히 클래스 생성자가 아니라, Nest의 DI 컨테이너가 인스턴스를 만들 때 필요한 의존성을 여기에 주입한다.

예:
```ts
@Injectable()
export class UserService {
  constructor(private readonly userRepository: UserRepository) {}
}
```

Nest가 `userRepository` 인스턴스를 생성해서 자동으로 넣어준다.

---

### 동작 방식 요약
1. 클래스에 `@Injectable()`, `@Controller()` 등이 붙어 있음
2. NestJS가 constructor의 파라미터를 보고 어떤 provider가 필요한지 분석함
3. DI 컨테이너에서 해당 provider를 찾아서 주입함

---

### 어디서 쓰이는가?

| 위치 | 용도 |
|------|------|
| Controller | 요청 처리 시 필요한 서비스 주입 |
| Service | DB 연결, 다른 서비스 주입 |
| Guard, Pipe, Interceptor | 유틸리티나 헬퍼 주입 |

@Module은 DI 대상 아님

---

### 접근자 예시
```ts
constructor(private userService: UserService) {}
```
→ 다음과 동일함:
```ts
private userService: UserService;

constructor(userService: UserService) {
  this.userService = userService;
}
```

---

### 주입 조건
1. 주입하려는 클래스가 `@Injectable()`이어야 함
2. 해당 provider가 모듈의 `providers`에 등록되어 있어야 함
3. 타입이 명확해야 Nest가 주입할 수 있음

---

### 사용 예시
```ts
@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get()
  getUsers() {
    return this.userService.findAll();
  }
}

@Injectable()
export class UserService {
  constructor(private readonly userRepository: UserRepository) {}

  findAll() {
    return this.userRepository.findAll();
  }
}
```

---

### 주의할 점
- 순환 참조 발생 시 `forwardRef()` 사용
- provider 누락 시 Nest can't resolve dependencies 에러 발생
- 타입 모호할 땐 `@Inject('TOKEN')` 데코레이터 사용 필요

---

### 요약

| 항목 | 설명 |
|------|------|
| 역할 | 의존성 주입 통로 |
| Nest가 자동 호출? | O |
| 필드 자동 선언 가능? | 접근자(private/public) 붙이면 가능 |
| 사용 위치 | Controller, Service, Guard, Pipe 등 |
| 주의사항 | 순환 참조, provider 누락, 타입 명시 필요 |
