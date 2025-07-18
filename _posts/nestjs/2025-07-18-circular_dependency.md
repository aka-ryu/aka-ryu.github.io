---
layout: post
title: NESTJS - 순환참조와 forwardRef()
category: NESTJS
---
#### 문제 상황
두 클래스가 서로를 주입하면 NestJS가 의존성 순서를 해결하지 못해 런타임 에러 발생.

```ts
// user.service.ts
@Injectable()
export class UserService {
  constructor(private readonly postService: PostService) {}
}

// post.service.ts
@Injectable()
export class PostService {
  constructor(private readonly userService: UserService) {}
}
```

#### 해결 방법: `forwardRef()` 사용
```ts
// user.service.ts
@Injectable()
export class UserService {
  constructor(
    @Inject(forwardRef(() => PostService))
    private readonly postService: PostService
  ) {}
}

// post.service.ts
@Injectable()
export class PostService {
  constructor(
    @Inject(forwardRef(() => UserService))
    private readonly userService: UserService
  ) {}
}
```

모듈에서도 `forwardRef` 필요:
```ts
// user.module.ts
@Module({
  imports: [forwardRef(() => PostModule)],
  providers: [UserService],
  exports: [UserService],
})
export class UserModule {}

// post.module.ts
@Module({
  imports: [forwardRef(() => UserModule)],
  providers: [PostService],
  exports: [PostService],
})
export class PostModule {}
```

---