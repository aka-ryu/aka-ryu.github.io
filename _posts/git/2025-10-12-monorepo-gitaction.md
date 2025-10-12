---
layout: post
title: GIT - 모노레포 구조에서 백엔드,프론트 효율적 배포
category: GIT
---




# GitHub Actions에서 프론트/백엔드 변경 감지 후 개별 배포하기

하나의 레포지토리(monorepo)에 프론트엔드(Next.js)와 백엔드(Nest.js)를 함께 관리할 때,
단순히 `main` 브랜치에 푸시될 때마다 전체를 재빌드/배포하면 불필요한 자원 낭비가 발생한다.

이를 해결하기 위해 **GitHub Actions의 `paths` 필터 기능**을 활용하여
각 폴더별 변경만 감지하고, 필요한 서비스만 자동 배포되도록 구성한 과정을 정리했다.

---

## ✅ 환경

* **레포지토리 구조**

  ```bash
  repo-root/
   ├─ front/   # Next.js 프론트엔드
   └─ back/    # Nest.js 백엔드
  ```
* **GitHub Actions 사용**

  * 각 디렉토리별 워크플로우 분리
  * 또는 하나의 워크플로우에서 조건부 실행
* **Node.js 버전**: 20.8.0
* **배포 환경**: EC2 (또는 Docker 기반)

---

## 1. 문제 상황

하나의 레포에서 프론트와 백엔드를 동시에 관리하면, 작은 수정이라도 전체 워크플로우가 실행된다.
예를 들어, 단순히 백엔드 로직만 변경해도 프론트 배포까지 다시 트리거되어 비효율적이다.

이를 해결하기 위해 GitHub Actions의 **경로 기반 트리거**(`on.push.paths`)를 활용한다.

---

## 2. 프론트엔드 전용 워크플로우 설정

`.github/workflows/frontend.yml`

```yaml
name: Deploy Frontend

on:
  push:
    branches:
      - main
    paths:
      - 'front/**'   # front 폴더 내 변경 시에만 실행
      - '.github/workflows/frontend.yml'

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: cd front && npm ci

      - name: Build
        run: cd front && npm run build

      - name: Deploy
        run: echo "✅ Deploy frontend..."
```

이렇게 설정하면 `front/` 폴더나 관련 워크플로우 파일이 수정된 경우에만 프론트엔드 빌드/배포가 실행된다.

---

## 3. 백엔드 전용 워크플로우 설정

`.github/workflows/backend.yml`

```yaml
name: Deploy Backend

on:
  push:
    branches:
      - main
    paths:
      - 'back/**'   # back 폴더 내 변경 시에만 실행
      - '.github/workflows/backend.yml'

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: cd back && npm ci

      - name: Build
        run: cd back && npm run build

      - name: Deploy
        run: echo "🚀 Deploy backend..."
```

이제 백엔드 코드만 수정해도 프론트엔드는 건드리지 않고, 백엔드만 자동으로 배포된다.

---

## 4. paths-filter로 조건 분기 통합 (선택)

만약 프론트와 백엔드를 하나의 YAML 파일로 통합하고 싶다면, `dorny/paths-filter` 액션을 이용할 수 있다.

```yaml
name: Conditional Deploy

on:
  push:
    branches:
      - main

jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      front: ${{ steps.filter.outputs.front }}
      back: ${{ steps.filter.outputs.back }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            front:
              - 'front/**'
            back:
              - 'back/**'

  frontend:
    needs: changes
    if: ${{ needs.changes.outputs.front == 'true' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying Frontend"

  backend:
    needs: changes
    if: ${{ needs.changes.outputs.back == 'true' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying Backend"
```

이 방식은 **하나의 워크플로우 파일로 프론트와 백엔드 배포를 동적으로 분기**시킨다.

---

## 🧠 정리

1. `on.push.paths` 옵션을 사용하면 폴더별 변경 감지로 불필요한 빌드를 막을 수 있다.
2. 프론트와 백엔드를 각각 다른 YAML로 분리하거나, `paths-filter`로 하나의 파일에 통합 가능하다.
3. 코드 변경 범위가 작아질수록 빌드/배포 시간과 비용을 줄일 수 있다.

이 구성을 적용하면, 모노레포 환경에서도 효율적인 CI/CD를 구현할 수 있다.
