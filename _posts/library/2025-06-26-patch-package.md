---
layout: post
title: LIBRARY - patch-package
category: LIBRARY
---

`patch-package`는 node_modules 내 설치된 라이브러리의 파일을 지속적으로 관리할 수 있게 해주는 도구다.

npm 또는 yarn으로 재설치가 이루어지면 node_modules 폴더가 초기화된다. 이때 직접 수정한 내용이 모두 사라지는데, `patch-package`를 사용하면 이를 손쉽게 유지하고 관리할 수 있다.


---

## patch-package 설치

```bash
npm install patch-package postinstall-postinstall
```

---

## package.json 파일 설정

```json
"scripts": {
  "postinstall": "patch-package"
}
```

> npm install 후에 patch-package 가 다음으로 바로 적용된다.

---

## 리버 수정

node\_modules/에서 수정 할 파일을 직접 변경한다.

```bash
vim node_modules/some-library/index.js
```

---

## patch 파일 생성

```bash
npx patch-package some-library
```

`patches/` 포맷에 파일이 생성된다.

---

## 목록 형식

```diff
--- a/node_modules/some-library/index.js
+++ b/node_modules/some-library/index.js
@@ -10,7 +10,7 @@
 function buggyFunction() {
-   return 'This is broken';
+   return 'This is fixed';
 }
```

---

## 테스트

npm install 후에 patch-package 가 자동으로 적용되며, node\_modules 변경 내용이 실제 다음 배포에도 모두 담겨집니다.

patches 포맷 파일 하나만 git 에 규정하면 협업자가 같이 가지고 가는 환경이 가능해진다.

---

## 주의 사항

* 버전이 변경되면 파일이 개별되기 때문에 patch 질적이 바로 적용 되지 않을 수 있다.
* 이때는 patch 파일을 다시 생성해야 한다.
* 조종한 patch가 없는 경우, 그 라이브러리에 포인트 수정을 가능한 가상 버거로 PR 보내는 것이 것이 가장 가장 효율적이다.

---

## 정보 요약

* node\_modules 변경은 `patch-package`로 관리하는 것이 최고의 방식
* postinstall 스크립트 가 필요
* patches 포맷을 git 에 포함해야 한다.

---

결국적으로 `patch-package`는 리버의 재설치 클릭을 담당할 수 있고, 일괄적으로 변경 관리를 간단하게 조정할 수 있다.
