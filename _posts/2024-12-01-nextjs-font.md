---
layout: post
title: next.js 에서 라이브러리 폰트가 안먹히는 이슈
subtitle: 폰트 제멋대로 먹힌다?
categories: Next
tags: [Next.js, 트러블슈팅]
image: /assets/images/20241201_font.png
---

### 😵 사건의 발단

---

우리가 지원하려는 React 기반의 디자인시스템 라이브러리에서는 모든 컴포넌트가 pretendard 폰트를 지원해야한다. 그래서 폰트 지원이 필요한 컴포넌트 스타일 파일 내에서 `@import"https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/static/pretendard.min.css";`
해당 임포트 구문을 추가해주고 있었다. 스타일도 우리가 의도한 대로 잘 먹었는데,,,그런데 말입니다.🫠

<br/>

라이브러리를 만들고, 해당 라이브러리를 소개하는 사이트를 Next.js 로 구축하였다. 그런데?
하나의 컴포넌트가 추가되고 사라지고에 따라 페이지의 폰트가 안먹는다?

<br/>

<img width="300" alt="폰트비정상" src="https://github.com/user-attachments/assets/e990062c-f230-4205-a308-eef81e2c9951">
<img width="300" alt="폰트정상" src="https://github.com/user-attachments/assets/5a8cc05e-ff65-42d9-8dcc-5fce177d215f">

<br/>
<br/>

### 원인 발견

---

개발자 모드에서 스타일 코드를 분석해보았다.
<img width="600" alt="스타일시트" src="https://github.com/user-attachments/assets/014981af-d818-489f-99b1-5c7ea2b8ab01">

<br/>

폰트가 먹히지 않아 문제가 발생하는 케이스에서는 이미지와 같은 에러가 났다. 빌드된 스타일 시트에서 import 구문이 하위에 있어 폰트가 불려와지지 않은 것이다.

구현하려는 Next 사이트에서 설정된 page 나 layout에 적용된 스타일이 우선 상단에 배치되고, 불러오는 라이브러리의 스타일이 하위에 적용되는 것을 확인할 수 있었다.

<br/>
<br/>

### 해결 방식

---

1. 사용자가 import 추가해주기
   그렇다면 import 구문의 위치를 상단으로 항상 고정하기 위한 방법이 필요했다. 하지만 그렇게 하기 위해선 사용자가 우리 라이브러리를 사용할때 여기에 import 추가해야 폰트 적용되요~ 하는 것도 이상하다...
   우선 근본적인 이슈인 import 구문이 하위로 밀려날 케이스가 없어야 하기 때문에 스타일 파일 내에서 폰트를 import 하는 일이 없어야 한다고 생각했다.

<br/>

2. 라이브러리에서 font 파일 가지고 있기
   import 방식이 아니라 폰트 파일 자체를 가지고 있고 경로 설정을 해준다면 딱히 문제가 없어 보였다. 하지만 번들 크기가 커지기도 하고 초기에 cdn으로 제공하는 폰트를 import 하는 방식을 채택했으니 이 방식을 유지하고 싶었다. 그래서 최후 3번의 방식을 택했다.

<br/>

3. 컴포넌트 호출 시 폰트 다운
   폰트 적용이 필요한 컴포넌트 사용 시, link 태그를 생성하여 폰트를 다운받자! 는 생각을 했다.

<br/>

```
const existingLink = document.querySelector(

'link[href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/static/pretendard.min.css"]',

);



if (!existingLink) {

const link = document.createElement("link");

link.rel = "stylesheet";

link.href =

"https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/static/pretendard.min.css";

document.head.appendChild(link);

}
```

<br/>

위의 코드와 같이 이미 폰트 링크가 존재하지 않을 때에만 폰트를 불러오는 링크를 생성하는 것이다. 기존에 있던 문제가 또하나 있긴 했는데 컴포넌트를 여러개 사용하면, 컴포넌트 css에 폰트 import 구문이 각각 있어 불필요한 import 가 빌드된 스타일 코드상 여러개가 찍혔다.
위의 방식이라면 몇개의 컴포넌트를 사용하던 불필요한 import 구문이 스타일 코드에 추가되지도 않을 테고, 예상치 못하게 폰트를 불러오지 못할 일이 거의 없을 것이다!
