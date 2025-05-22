---
layout: post
title: ISSUE - 리액트네이티브 안드로이드 지원기기 0 이슈
category: ISSUE
---

# React Native 앱 업데이트 후 '지원 기기 0대' 문제 원인 및 해결법

React Native로 배포 중인 앱에서 업데이트 후 Google Play Console에 '지원되는 기기 0대'로 표시되는 문제가 발생했다. 이전까지는 문제없이 수많은 기기에서 지원되었지만, 특정 버전 업데이트 이후 이 현상이 발생했다.

## 원인 요약

문제의 핵심은 AndroidManifest.xml 내 `<uses-feature>` 태그 설정과 Google Play의 기기 필터링 정책 변화 때문이다.

### 1. `<uses-feature>`의 기본 동작 변화

`<uses-feature>`에서 `android:required` 속성을 명시하지 않으면, 예전에는 Google Play가 이를 선택적(optional)으로 간주하는 경우가 많았다. 하지만 최근에는 해당 속성이 생략된 경우 **무조건 필수(required=true)** 로 간주하는 정책으로 변경되었다.

예시:

```xml
<!-- 문제를 일으킨 선언 -->
<uses-feature android:name="android.hardware.locale" />

<!-- 해결 방법 -->
<uses-feature android:name="android.hardware.locale" android:required="false" />
```

### 2. OpenGL ES 버전 필수 지정

```xml
<uses-feature android:glEsVersion="0x00020000" android:required="true" />
```

이 설정은 OpenGL ES 2.0 이상을 요구하는 것으로, 이를 지원하지 않는 기기는 자동으로 필터링된다. 최신 앱에서는 일반적으로 필요한 설정이지만, 구형 기기나 에뮬레이터 등은 이 조건으로 인해 제외될 수 있다.

### 3. Google Play 기기 필터링 정책 강화

2024년 하반기부터 Android 14 대응 정책과 함께 Google Play의 앱 심사 및 기기 필터링 기준이 엄격해졌다. 다음과 같은 상황에서 앱이 호환되지 않는 것으로 간주될 수 있다:

* `uses-feature`에서 `required` 속성이 빠져있음
* 외부 라이브러리나 빌드 도구에서 자동으로 `uses-feature`가 추가됨
* 권한 요청(permission)으로 인해 암묵적인 `uses-feature`가 추가됨

## 해결 방법

### 1. `android:required="false"` 명시

앱이 특정 하드웨어를 사용하지만 필수는 아닌 경우, 반드시 `required="false"`를 명시해야 한다.

```xml
<uses-feature android:name="android.hardware.camera" android:required="false" />
```

### 2. 필요 없는 `<uses-feature>` 제거 또는 완화

직접 명시하지 않았더라도 라이브러리를 통해 추가되었을 수 있으니 build.gradle 및 AndroidManifest.xml을 모두 점검해야 한다.

### 3. glEsVersion 설정 점검

OpenGL ES 버전이 필수일 필요가 없다면 아래처럼 변경하거나 삭제한다:

```xml
<uses-feature android:glEsVersion="0x00020000" android:required="false" />
```

## 결론

이 문제는 코드 상의 문제라기보다는 Google Play의 정책 변화로 인해 발생한 것으로, `<uses-feature>` 선언을 정확히 관리하지 않으면 지원 기기가 0대가 되는 심각한 문제가 발생할 수 있다. 배포 전에는 Google Play Console에서 '호환 기기 수'를 꼭 확인해야 하며, manifest 파일의 하드웨어 요구사항을 항상 명시적으로 선언하는 습관이 필요하다.

