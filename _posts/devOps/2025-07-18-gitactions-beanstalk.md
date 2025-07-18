---
layout: post
title: DEVOPS - 거래소 지갑 테이블 모델링
category: DEVOPS
---
회GitHub Actions와 AWS Elastic Beanstalk을 조합하면 꽤 단순하고 강력하게 CI/CD를 구성할수 있다.
---

## 전체 흐름 요약

```
1. GitHub에 코드 Push
2. GitHub Actions에서 워크플로우 실행
3. 빌드 & 테스트 진행
4. 결과물을 zip으로 묶어 S3에 업로드
5. Elastic Beanstalk에 새 버전 등록 및 배포 요청
6. Beanstalk가 자동으로 앱을 새 버전으로 반영
```

---

## 구성 요소 정리

| 구성 요소 | 역할 |
|-----------|------|
| GitHub Actions | 빌드, 배포 자동화 |
| AWS Elastic Beanstalk | 배포 대상 플랫폼 (PaaS) |
| AWS S3 | 배포 파일(zip) 임시 저장소 |
| AWS IAM | GitHub에서 AWS 접근 허용 (Access Key 필요) |

---

## GitHub Actions 설정 예시

`.github/workflows/deploy.yml` 파일 생성

```yaml
name: Deploy to Elastic Beanstalk

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18

      - name: Install dependencies
        run: npm install

      - name: Build app
        run: npm run build

      - name: Zip build files
        run: zip -r app.zip . -x '*.git*' 'node_modules/*' 'tests/*'

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-2

      - name: Upload to S3
        run: aws s3 cp app.zip s3://my-eb-bucket/app.zip

      - name: Deploy to Elastic Beanstalk
        run: |
          aws elasticbeanstalk create-application-version \
            --application-name my-app \
            --version-label v-${{ github.run_number }} \
            --source-bundle S3Bucket=my-eb-bucket,S3Key=app.zip

          aws elasticbeanstalk update-environment \
            --environment-name my-env \
            --version-label v-${{ github.run_number }}
```

---

## AWS 측 사전 준비

1. **IAM 사용자 생성**
   - `AmazonS3FullAccess`, `AWSElasticBeanstalkFullAccess` 권한 부여
   - Access Key, Secret Key 발급 → GitHub에 저장

2. **S3 버킷 생성**
   - 예: `my-eb-bucket`

3. **Elastic Beanstalk 앱과 환경 생성**
   - 예: `my-app`, `my-env`

---

## GitHub Secrets 설정

| Key | 설명 |
|-----|------|
| `AWS_ACCESS_KEY_ID` | IAM에서 받은 액세스 키 |
| `AWS_SECRET_ACCESS_KEY` | IAM에서 받은 시크릿 키 |

GitHub Repository → Settings → Secrets 에 등록해두면 된다.

---

## 폴더 구조 예시 (Node.js 기준)

```
my-app/
├── .github/workflows/deploy.yml
├── app.js
├── package.json
├── Procfile            # 예: web: node app.js
├── .ebextensions/      # 선택 사항 (Elastic Beanstalk 설정 파일)
```

---

## 참고 사항

- `github.run_number`를 이용해서 매 배포마다 고유한 버전 라벨을 생성할 수 있음
- `aws elasticbeanstalk create-application-version` 명령어가 핵심
- S3에 올린 zip을 Beanstalk이 직접 받아서 배포함
- 문제가 생기면 AWS 콘솔에서 이전 버전으로 쉽게 롤백 가능

---

## 마무리

이 방식은 서버 없이도 깔끔하게 배포를 자동화할 수 있어서, 사이드 프로젝트나 소규모 서비스 운영에 특히 유용하다. EC2나 Docker를 쓰는 경우보다 훨씬 간단하게 CI/CD 파이프라인을 꾸릴 수 있다는 점이 장점이다.
