---
layout: post
title: NESTJS - 모듈 자체 export vs 서비스만 export 차이
category: NESTJS
---

#### 모듈 자체 export (`exports: [SomeModule]`)
- 해당 모듈이 export한 모든 provider를 함께 외부로 전달함
- 일종의 패스스루(pass-through) 역할
- 예:
  ```ts
  @Module({
    imports: [AModule],
    exports: [AModule]
  })
  export class BModule {}
  ```
- AModule이 export한 모든 provider(AService 등)를 CModule에서 사용 가능

#### 서비스만 export (`exports: [SomeService]`)
- 딱 필요한 provider만 외부에 노출함
- 더 명확하고 의도가 분명함
- 예:
  ```ts
  @Module({
    imports: [AModule],
    exports: [AService]
  })
  export class BModule {}
  ```
- AModule에 다른 export가 생겨도 영향을 받지 않음

#### 요약
| 항목 | 모듈 자체 export | 서비스만 export |
|------|-------------------|------------------|
| 의미 | 모듈이 export한 걸 통째로 전달 | 특정 provider만 전달 |
| 제어 범위 | 넓음 (모든 provider 포함) | 좁음 (명시한 것만) |
| 가독성 | 간결 | 명확 |
| 사용 추천 | 공통 모듈 전달할 땐 편함 | 실무에서는 이 방식이 더 안정적 |

필요한 provider만 선택적으로 외부에 전달하고 싶을 땐 `서비스만 export` 하는 게 좋고, 여러 개를 한번에 넘겨야 할 때는 `모듈 자체 export`도 쓸 수 있음.
