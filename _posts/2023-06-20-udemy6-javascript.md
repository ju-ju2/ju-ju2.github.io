---
layout: post
title: 유데미x스나이퍼팩토리 10주 완성 프로젝트 캠프 15일차 - javascript
subtitle: javascript로 랜덤 배경이미지, 시계 만들기
categories: javascript
tags: [udemy, javascript]
image: /assets/images/udemy/udemy.png
---

html과 css 이후 javascript 실습시간을 가지고 있다. 이번 과제는 아래와 같다.

1. vanilla js로 배경이미지 랜덤 변경
2. vanilla js로 인사 만들기
3. vanilla js로 시계 만들기

<br />
우선 1번을 수행하기 위해 마음에 드는 배경 5가지를 골라 이미지의 주소를 배열로 정렬하였다. 무료 배경이미지는 [unsplash](https://unsplash.com/ko){: target="\_blank" } 에서 픽픽!😎

```javascript
const images = [
  "https://images.unsplash.com/photo-1567095761054-7a02e69e5c43?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8MTR8fGNvbG9yfGVufDB8fDB8fHww&auto=format&fit=crop&w=800&q=60",
  "2",
  "3",
  "4",
  "5",
];
```

해당 1번 과제에서는 `Math.random()`, `Math.floor()`가 핵심이다.

<h4 id="includes-reduce" style="color: #ff5100;;">Math.random()</h4>

---

0 이상 1 미만의 임의의 숫자를 반환한다.

```javascript
randomNumber = Math.random();
console.log(randomNumber);

// 0.461564007606188
```

<h4 id="includes-reduce" style="color: #ff5100;;">Math.floor()</h4>

---

소수점 이하를 제거하여 정수 부분만 남기는 내림함수이다.

```javascript
Math.floor(5.9);
// 5
Math.floor(5.1);
// 5
Math.floor(5.253355);
// 5
```

<br />
그리고 이 둘 함수를 응용하면 0부터 4까지의 랜덤한 숫자를 얻을 수 있다.

```javascript
Math.floor(Math.random() * 5);
```

그리고 위의 랜덤으로 얻어진 숫자를 배열 인덱스에 매칭시켜주면 끝이다.

```javascript
const randomImg = images[Math.floor(Math.random() * 5)];
const backgroundImg = `url(${randomImg})`;
document.body.style.backgroundImage = backgroundImg;
```

0부터 4까지의 임의의 인덱스를 가진 이미지가 body의 background 소스가 되어 나타난다. 새로고침할 때다 변경되는 이미지 확인 완료!

<img width="400" alt="랜덤 배경" src="https://github.com/ju-ju2/precamp_class/assets/71650663/36216abf-ae4f-498e-98e0-4ca4d8545145">

<br />
<br />
<br />
<br />
그 다음 2번 과제인 인사추가 부분은 아래의 코드로 추가해주었다.

```javascript
const hello = document.createElement("h1"); // h1태그 생성
hello.className = "hello"; // class 속성 적용
hello.innerText = "hello"; // 텍스트 생성
document.body.appendChild(hello); // body에 태그 추가
```

<br />
<br />
<br />
<br />
그리고 마지막 3번 과제, 시간 생성부분이다.

자바스크립트에서 제공해주는 내장 함수 중 `new Date()`와 이와 관련된 date함수를 이용할 것이다.

<h4 id="includes-reduce" style="color: #ff5100;;">new Date()</h4>

---

```javascript
new Date();
// Tue Jun 20 2023 14:18:50 GMT+0900 (한국 표준시)
```

위처럼 new Date() 함수는 현재 날짜와 시간을 나타내는 Date 객체를 생성한다. 해당 객체를 변수로 받아와 시계를 만드는데 필요한 시, 분, 초 를 생성해보자.

```javascript
const date = new Date(); // 실시간 날짜 생성
const hours = date.getHours(); // 시간 가져오기
console.log(hours); // 14

const min = date.getMinutes(); // 분 가져오기 // 18
const sec = date.getSeconds(); // 초 가져오기 // 50

// const clock = document.querySelector(".clock");
clock.innerText = `${hours}:${min}:${sec}`;
```

위와 같이 `getHours`, `getMinutes`, `getSeconds`함수는 Date 객체에서 현재 시간의 시간, 분, 초 부분을 가져오는 메서드이다. 이를 활용하여 시계를 만들어 줄 수 있다.  
하지만 해당 객체는 페이지가 랜딩 될 때에만 나타나기 때문에 현재 시간을 보여줄 뿐, 시계의 역할을 하기엔 아직 애매해다.  
그렇기 때문에 `setInterval`이라는 메서드를 이용하여 1초에 한번씩 화면을 업데이트해주면 된다.

아래는 전체 코드이다.

```javascript
const clock = document.querySelector(".clock");

function getClock() {
  const date = new Date();
  const hours = date.getHours();
  const min = date.getMinutes();
  const sec = date.getSeconds();
  clock.innerText = `${hours}:${min}:${sec}`;
}

getClock();
setInterval(getClock, 1000);
```

<img width="400" alt="시간 한자리" src="https://github.com/ju-ju2/precamp_class/assets/71650663/ff841c70-d73e-4dad-a291-47dd81af4662">

그런데 말이다. 뭔가 부족하다. 시간이 잘 가고 있지만 시, 분, 초가 한자릿수일때 01초가 아닌 1초로 보여진다. 모든 시간을 2자릿수로 만들어주기 위한 작업이 더 필요하다. 아래의 padStart() 메서드를 통해 자릿수를 지정해주자
<br />
<br />

<h4 id="includes-reduce" style="color: #ff5100;;">padStart()</h4>

---

padStart() 메서드는 다음과 같은 구문을 가진다.  
`시작할 문자열.padStart(목표 길이, 시작할 문자열);`  
해당 매서드는 `문자열`에서만 적용되기 때문에 시계를 만드는데 사용되는 number 타입을 string 타입으로 우선 바꿔줘야한다. 그리고 0으로 시작되는 두자릿 수를 만들기 위해 인자로 2와 0을 대입하면 된다. 아래는 수정된 코드와 결과이다.

```javascript
const clock = document.querySelector(".clock");

function getClock() {
  const date = new Date();
  const hours = String(date.getHours()).padStart(2, 0);
  const min = String(date.getMinutes()).padStart(2, 0);
  const sec = String(date.getSeconds()).padStart(2, 0);
  clock.innerText = `${hours}:${min}:${sec}`;
}

getClock();
setInterval(getClock, 1000);
```

<img width="400" alt="시간 두자리" src="https://github.com/ju-ju2/precamp_class/assets/71650663/a9ab9815-b527-4140-9615-23f730ef406c">

깔끔쓰👍🏻
<br /><br /><br /><br />

본 후기는 유데미-스나이퍼팩토리 10주 완성 프로젝트캠프 학습 일지 후기로 작성 되었습니다. #프로젝트캠프 #프로젝트캠프후기 #유데미 #스나이퍼팩토리 #웅진씽크빅 #인사이드아웃 #IT개발캠프 #개발자부트캠프 #리액트 #react #부트캠프 #리액트캠프
