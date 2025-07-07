---
layout: post
title: INTERVIEW - 면접 질문 복습
category: INTERVIEW
---

## 1. nestjs 클라이언트에서 요청이 들어온 경우 라이프싸이클


1. **미들웨어(Middleware)**
   - 가장 먼저 실행
   - 요청(req) 전처리, 로깅, 인증 등

2. **가드(Guard)**
   - 인증/권한 체크
   - true면 다음 단계로 진행

3. **인터셉터(Interceptor)**
   - 요청/응답 가로채서 변형, 로깅, 캐싱, 응답 포맷 등

4. **파이프(Pipe)**
   - 데이터 유효성 검사, 변환 (DTO)

5. **컨트롤러(Controller)**
   - 라우팅/핸들러 실행 (비즈니스 로직 진입점)

6. **서비스/프로바이더(Service/Provider)**
   - 실제 비즈니스 로직/DB 처리

7. **익셉션 필터(Exception Filter)**
   - 에러 발생 시 캐치, 응답 변환

<br><br>

---

<br><br>

## 2. UserGaurd 의 작동순서

1. 클라이언트가 요청을 보냄
2. 미들웨어가 실행됨
3. **User Guard의 `canActivate()` 함수가 실행되어**
   - 토큰/세션 등 인증 검사
   - 권한 확인
   - true면 다음 단계로 진행, false나 예외면 요청 거부
4. 파이프/인터셉터 등 후속 처리
5. 컨트롤러 실행
