---
layout: post
title: DB - 거래소 지갑 테이블 모델링
category: DB
---

거래소를 개발하면서 초기 DB설계를 진행중인데 고민사항이 하나 생겼다.  
a,b,c 거래소에서 링크된 유저의 자산을 받아와서 DB에 저장하려고 하는데 다음과 같이 체크사항들이 존재했다.

- 거래소 API 응답 양식은 거래소별로 다르다.
- 거래소에서 새로운 코인이 상장되거나 폐지될수 있다.
- 모든 거래소 API는 보유중인 코인 데이터만 보내주었다.

코인이 추가되거나 삭제될수도 있고, 보유중이지 않은 코인에 대해서도 0 or NULL 로 데이터를 들고 있자니 DB스토리지 공간도 낭비될것 같았다.  
<br>
그래서 생각한게 API 응답데이터를 JSON문자열로 저장해두고 필요한 페이지 접근할때마다 파싱해서 사용할까도 생각했었는데  
또 문제되는 부분이 해당 데이터는 읽고, 수정하는 경우가 많아 사용할때마다 백엔드자원으로 파싱을 해야하는데 그로인한 부담도 늘어난다는것  
(유저입금,출금,거래로인한 자산변동)
<br><br>

그래서 한참을 고민하고있는데 엥?... <br><br>
생각해보니 애초에 거래소에 링크된 블록체인 지갑이기 때문에 내 서비스 DB와 실시간으로 연동되지 않기 때문에  
사용할때마다 거래소 API 로 실시간 데이터를 받아와서 내 DB와 싱크를 맞춰야하네?<br><br>

조회,수정을 하려고 하면 일단 실시간 싱크부터 체크해야 되기 때문에 무조건 거래소 API로 실시간 데이터를 받아야 하고  
그렇다는건 코인마다 컬럼으로 저장을 한다고 하더라도 결국엔 무조건 실시간 응답받은 데이터를 파싱,분해,해석 하는 로직이 필요하네?<br><br>

고로, JSON문자열로 저장하는게 맞는거 같다.