---
layout: post
title: 유데미x스나이퍼팩토리 10주 완성 프로젝트 캠프 17일차 - javascript2
subtitle: javascript로 날씨 데이터 받아오기
categories: javascript
tags: [udemy, javascript]
image: /assets/images/udemy/udemy.png
---

javascript 두번째 과제는 날씨 데이터를 받아오는 것이다. 오늘은 날씨 데이터를 받아와 해당 날씨에 따른 배경이미지를 매칭하는 과정을 포스팅 해보겠다. 과정은 아래와 같다.

<br />

<h4 style="color: #ff5100;;">1. navigator로 위도 경도 가져오기</h4>

---

날씨를 알려면, 내가 있는 위치를 먼저 알아야한다. `navigator` 라는 자바스크립트 내장함수로 나의 위치를 불러오고 변수에 담아주자.

```javascript
function geolocation() {
  navigator.geolocation.getCurrentPosition(handleSuccess, handleError);
}

// 함수실행
geolocation();

// 성공 로직
function handleSuccess(data) {
  const latitude = data.coords.latitude;
  const longitude = data.coords.longitude;
}

// 실패 로직 (위치 권한 설정이 꺼져있으니 실패 함수가 호출되었다.)
function handleError() {
  alert("실패");
}
```

<br /><br />

<h4 style="color: #ff5100;;">2. 날씨 API 가져오기</h4>

---

[openweathermap](https://openweathermap.org/){: target="\_blank" }이라는 사이트에서 날씨 API를 무료로 제공해준다. 회원가입을 하면 key를 주는데 해당 키를 가지고 API에 적용해주면 된다. 나는 이번에 `Current Weather Data`를 사용했다.

해당 API 주소는 **https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${APIkey}** 이며, 값을 변수로 설정해도 되고, 하드코딩해줘도 된다. 나는 위의 `geolocation()` 함수가 실행되며 받은 위도, 경도 값을 변수로 넣어줄 것이다.

```javascript
function getWeather(latitude, longitude) {
  const lat = latitude;
  const lon = longitude;
  const APIkey = "본인의 API KEY";
  fetch(
    `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${APIkey}`
  )
    .then((res) => {
      return res.json();
    })
    .then((res) => {
      console.log(res);
      // 배경이미지 설정 로직
    });
}
```

<br />

위의 코드로 받은 데이터 로그를 찍어보면 아래와 같다.

<img width="400" alt="날씨 api 데이터값" src="https://github.com/ju-ju2/precamp_class/assets/71650663/ded7b3bd-3419-4a52-8b88-8b72eaaa807e">

나는 여기서 weather 배열의 0번째 객체의 main 값을 가져와 배경을 설정해 줄 것이다.

<br /><br />

<h4 style="color: #ff5100;;">3. 이미지 지정하기</h4>

---

우선 데이터 값에 맞는 이미지 주소가 필요하다. 받아올 main 이라는 data 값에는 많은 종류가 있을 것이다. 하지만 나는 **Clear, Rain, Drizzle, Clouds, Mist** 5가지로만 지정해주고 5가지 배경만 준비했다. (찾아보니 Drizzle은 부슬비라 rain과 같은 이미지를 지정했다.)

```javascript
// 이미지 주소 변수에 담아주기
const rainUrl = "이미지 주소";
const cloudsUrl = "이미지 주소";
const clearUrl = "이미지 주소";
const mistUrl = "이미지 주소";

// getWeather() 코드 추가
function getWeather(latitude, longitude) {
  // ...
    .then((res) => {
      console.log(res);
      let imgSrc;
      const weather = res.weather[0].main;
      if (weather.includes("Clear")) {
        imgSrc = clearUrl;
      } else if (weather.includes("Rain")) {
        imgSrc = rainUrl;
      } else if (weather.includes("Drizzle")) {
        imgSrc = rainUrl;
      } else if (weather.includes("Clouds")) {
        imgSrc = cloudsUrl;
      } else if (weather.includes("Mist")) {
        imgSrc = mistUrl;
      } else {
        imgSrc =
          "기본 이미지 주소";
      }

// div태그 속성(이미지, 텍스트) 지정
      const weatherImg = document.querySelector(".weather");
      weatherImg.style.backgroundImage = `url(${imgSrc})`;
      weatherImg.textContent = weather;
    });
}
```

<br />
그리고 브라우저 창을 확인해 보면 짠!😎

<img width="400" alt="날씨 api 데이터값" src="https://github.com/ju-ju2/precamp_class/assets/71650663/1adbf007-4bfe-45ea-a526-f631dbeea242">

<br />

---

사실 API 받아오는 속도가 많이 느려 처음에는 통신이 안되나? 생각했다. 네트워크창을 살피려던 찰나에 이미지가 짠하고 나타났다. 이미지가 나타나기전 로딩 스피너를 추가하는 것도 UX적으로 좋을 것 같다. 여튼 오늘의 포스팅 끝!

<br /><br /><br /><br />

본 후기는 유데미-스나이퍼팩토리 10주 완성 프로젝트캠프 학습 일지 후기로 작성 되었습니다. #프로젝트캠프 #프로젝트캠프후기 #유데미 #스나이퍼팩토리 #웅진씽크빅 #인사이드아웃 #IT개발캠프 #개발자부트캠프 #리액트 #react #부트캠프 #리액트캠프
