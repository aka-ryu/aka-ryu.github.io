---
layout: post
title: BLOCKCHAIN - 클레이튼 멀티센더 구현
categories: BLOCKCHAIN
---

## 프로젝트 설명

합류 예정인 팀에서 이벤트로 참가자들에게 클레이튼 네트워크의 특정 코인을 나눠주어야 하는데 이때 참여자들 한명 한명에게 단일 트랜잭션을 보내는것 보다  
csv 파일로 유저리스트를 올려 한번의 멀티샌딩 트랜잭션으로 수수료도 절감하고 편리함도 갖추기 위해 개발하였다.  
(이더리움,바이낸스,폴리곤 등의 네트워크는 이미 다른곳에서 개발하여 이용할수 있는 곳이 많지만 클레이튼은 없다.)
<br>

## 프로젝트 구성

FE - react  
HS - vercel  
REPO - <a href="https://github.com/aka-ryu/multisender " target="_blank">https://github.com/aka-ryu/multisender</a>  
domain - <a href="https://multisender-xodd.vercel.app/" target="_blank">https://multisender-xodd.vercel.app/</a>  
외, 이용자는 크롬 익스텐션 kaikas 지갑을 설치해야 한다.

## 프로젝트 설명

<img src="/public/img/20240723_03.png" alt="screenshot" width="1000" height="600">
<img src="/public/img/20240723_04.png" alt="screenshot" width="1000" height="600">
<img src="/public/img/20240723_05.png" alt="screenshot" width="1000" height="600"><br>
<img src="/public/img/20240723_06.png" alt="screenshot" >
<img src="/public/img/20240723_07.png" alt="screenshot" ><br>
<img src="/public/img/20240723_08.png" alt="screenshot" >

## 프로젝트 과정, 특이사항

개발하는데 까지는 총 3일정도 소요되었다.  
블록체인 트랜잭션 관련 개발은 여러번 진행 했기 때문에 <a href="https://github.com/aka-ryu/ryu-s-wallet" target="_blank">(이전 블록체인 토이프로젝트 앱)</a>  
블록체인과 해당 트랜잭션 관련 로직에서의 어려움은 크게 없었지만 개발중 이용했던 <a href="https://github.com/klaytn/caver-js" target="_blank">caver.js</a>(kaikas지갑 헬프모듈)가
업데이트가 거의 중단된 상태라는 점, 기타 [react-script5, webpack5, polyfill](/react/2024/07/23/polyfill.html)
<br><br>
개발자체에는 크게 문제가 되거나 기술적 어려움이 있는부분이 없었지만 클레이튼 네트워크 자체가 많이 죽은건지
로직이 네트워크 상태에 영향을 받는 부분이 존재하였고  
페이지에서 잔액조회가 일시적으로 안되는등 kaikas 혹은 네트워크 상태에 따라 기능이 작동을 했다가 하지 않았다가 하는 특이사항이 보였고  
그로인해 예상 수수료 예측에 대해서는 건너뛰고 진행하였다.  
<br><br>
급하게 진행한거라 트랜잭션 전파의 여러 단계마다의 에러핸들링 처리가 되어있지 않지만 이벤트 일정을 정상적으로 완료하였고  
현재는 프라이빗 BO 에서 코드안정화, 기능추가, 메타마스크 방식 추가 등의 작업을 진행중이다.
