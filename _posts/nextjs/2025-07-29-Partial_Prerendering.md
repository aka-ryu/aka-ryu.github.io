---
layout: post
title: NEXTJS - 부분 사전 렌더링
categories: NEXTJS
---
# Next.js - 부분 사전 렌더링 (Partial Static Generation) 정리

Next.js에서는 페이지 전체를 정적으로 미리 생성하는 것뿐만 아니라, **일부만 정적으로 만들고 나머지는 요청 시 생성하는 방식**도 지원한다.
이걸 **부분 사전 렌더링**이라고 부른다.

---

## ✅ 개념

> 중요한 일부 데이터는 **빌드 시점에 정적으로 생성**하고,
> 나머지는 **요청이 올 때 동적으로 처리하는 방식**이다.

이 방식은 특히 **동적 경로가 많고**, **데이터가 계속 추가되는 페이지**에 유용하다.

---

## ✅ 어떻게 구현하나?

### Pages Router에서:

```ts
// pages/blog/[slug].tsx

export async function getStaticPaths() {
  const slugs = await getPopularPostSlugs();
  return {
    paths: slugs.map(slug => ({ params: { slug } })),
    fallback: true, // 나머지는 요청 시 생성
  };
}

export async function getStaticProps({ params }) {
  const post = await fetchPost(params.slug);
  return {
    props: { post },
    revalidate: 60, // ISR: 60초마다 백그라운드에서 재생성
  };
}
```

### App Router에서도 비슷하게:

```ts
// app/blog/[slug]/page.tsx

export async function generateStaticParams() {
  const slugs = await getPopularPostSlugs();
  return slugs.map(slug => ({ slug }));
}

export const dynamicParams = true; // fallback true 역할
export const revalidate = 60; // ISR
```

---

## ✅ 각각의 의미

| 키워드                    | 설명                               |
| ---------------------- | -------------------------------- |
| `getStaticPaths`       | 어떤 경로를 미리 정적으로 생성할지 지정함          |
| `fallback: true`       | 나머지 경로는 **요청이 왔을 때** 서버에서 생성됨    |
| `getStaticProps`       | 각 경로의 데이터 가져옴 (정적 생성용)           |
| `revalidate`           | 일정 시간마다 정적 페이지를 **백그라운드에서 재생성**함 |
| `generateStaticParams` | App Router에서 정적으로 만들 경로 정의       |

---

## ✅ 예시 상황

* 인기 블로그 글은 빌드 시점에 미리 정적 생성함
* 나머지 글은 요청이 왔을 때 생성하고, 다음 요청부터는 캐시에서 제공함
* 최신성도 유지하기 위해 일정 주기로 자동 갱신되게 설정함

---

## ✅ 왜 쓰는가?

* 모든 페이지를 다 정적으로 만들면 빌드 시간이 오래 걸릴 수 있다.
* 잘 안 쓰는 페이지까지 미리 만들 필요는 없다.
* 필요한 페이지만 빠르게 만들고, 나머지는 유연하게 처리하는 게 효율적이다.

---

## ✅ 요약

* 부분 사전 렌더링은 **정적 속도 + 동적 유연함**을 동시에 잡기 위한 기능이다.
* `fallback`, `revalidate`, `dynamicParams` 등을 통해 구현할 수 있다.
* Next.js의 ISR과 함께 쓰면 성능과 최신성을 모두 챙길 수 있다.
