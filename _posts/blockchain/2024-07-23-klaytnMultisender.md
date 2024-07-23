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

<img src="/public/img/20240723_03.png" alt="screenshot" width="600" height="400">

## 해결방안

마지막 국어 데이터를 추가하는 로직을 제거하고 국어 과목에 교육부가 아닌 다른 출판사의 책이 연결되지 않도록 다른부분을 점검
