---
layout: post
title: REACTNATIVE - 네이버 oauth2 인증 이슈
category: REACTNATIVE
---

## 개요

특정 유저가 네이버로 로그인하기 이벤트를 호출하여도 아무런 반응이 없는 이슈가 발생하였다.
<br>

## 원인

에러 로그 내용  
BUG IN CLIENT OF UIKIT: The caller of UIApplication.openURL(_:) needs to migrate to the non-deprecated UIApplication.open(_:options:completionHandler:). Force returning false (NO).  
<img src="/public/img/20241106/20241106_01.png" alt="screenshot" width="500">
<img src="/public/img/20241106/20241106_02.png" alt="screenshot" width="500">
<img src="/public/img/20241106/20241106_03.png" alt="screenshot" width="500">

## 해결방안

해당 모듈의 4.0.2 릴리즈에서 이슈가 픽스되었음으로 해당 모듈의 버전을 업데이트
