---
layout: post
title: ISSUE - NextJS 에서 env 환경 분리가 되지 않는점.
category: ISSUE
---

## 개요

로컬, 개발계, 운영계 3가지 환경을 운영하며  
.env.local, .env.dev, .env.prd 3개의 env를 이용하려 했으나  
next 자체에서 env 우선권을 가지고 env-cmd, dotenv-cli 모듈을 사용해도 무시하는 상황

## 원인

nextjs 에서는 실행 환경에 따라 env를 자동으로 주입한다.  
env-cmd, dotenv-cli 로 환경별 env를 사용하는 방법이 있었으나 버전이 업데이트되며 막혔다.

## 해결방안

![screensh](/public/img/20240807/20240807_00.png)  
왼쪽이 기존 스크립트, 오른쪽이 변경 후 스크립트
env파일들을 루트경로가 아닌 곳에 배치하여 강제로 배정하면 된다.
