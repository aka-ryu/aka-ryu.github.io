---
layout: post
title: HTML,CSS,JS - 그라디언트 인풋보더
category: HTML,CSS,JS
---

최근에 진행했던 프로젝트에서 그라디언트 보더를 인풋을 만들 상황이 생겨서 만들었던것 복습

방법을 먼저 말하자면 인풋 보더에는 단색만 가능하다.

그래서 먼저 그라디언트 배경을 만들어두고

그안에 input을 넣어서 만들면 된다. ! <br><br>

<img src="/public/img/20241004/20241004_00.png" alt="screenshot" >
<img src="/public/img/20241004/20241004_01.png" alt="screenshot" ><br><br>

하지만 만약에 기본상태에서는 검은색 이었다가
focus 시에만 그라디언트가 나와야 한다면 살짝 복잡해진다.<br><br>

역시나 방법을 미리 설명해주자면<br><br>
일단 그라디언트 div를 밑에 깔아두고 그 위에 input 을 덮는다.<br>
이때 input의 크기(border두깨 포함)와 div사이즈가 똑같으면 그라디언트 div가 보이지 않는다.<br>
(border 두깨 계산을 잘해야 한다 border 두깨에 따라 인풋이 살짝 이동하는 느낌이 들거나 placeholder 들이 이동하는 느낌이 들게된다.)<br><br>

<img src="/public/img/20241004/20241004_02.png" alt="screenshot" >
<img src="/public/img/20241004/20241004_03.png" alt="screenshot" ><br><br>

focus 시에 input의 width 와 height 을 인풋크기만큼 줄이고 가로세로 중앙정렬 및 border를 none으로 바꾼다<br><br>

<img src="/public/img/20241004/20241004_04.png" alt="screenshot" >
<img src="/public/img/20241004/20241004_05.png" alt="screenshot" ><br><br>

이때 지금 기본 html, css 에서는 input 의 크기,높이 안에 border 도 포함되어서 계산되지만<br>
chakraUI 같은 UI프레임워크를 사용시에는 border가 포함되지 않는 케이스도 있어서 <br>
확인하면서 focus 시에 padding 등의 조절도 해줘야 placeholder 나 내부 텍스트가 움직이지 않는다.<br><br>

끗!
