---
layout: post
title: Link와 useNavigate (feat.SPA)
subtitle: 무엇이 무엇이 똑같을까
categories: react
tags: [react]
image: /assets/images/2023-26-26-link/link.png
---

#### 들어가며😃

---

리액트를 사용하다 보면 페이지 이동(화면 전환)이 빈번하게 일어난다. 이때 `Link` 컴포넌트를 통해 이동하는 방식, `useNavigate()` 함수를 통해 이동하는 방식이 대표적일 것 이다. 오늘은 이 두 방식에 대해 알아보고 비교해보겠다.

<br />

<em style='font-size: 20px; color: #BAD1C2; font-weight: bold;'>앗! 그전에! SPA?!🤷🏻‍♀️</em>

리액트는 `SPA(Single Page Application)`로 전체 프로젝트가 하나의 페이지로 구성되어있다. 그렇기 때문에 페이지를 이동할 수가 없다. 하지만 `React Router`를 통해 익숙히 우리가 해왔듯 페이지를 이동할 수 있다. React Router는 리액트 애플리케이션의 라우팅을 관리하기 위한 라이브러리로, 주소(url)과 컴포넌트를 연결하여 페이지 전환을 구현한다. 자 그럼 React Router가 지원하는 기능 중 Link 를 먼저 알아보자.
<br /><br />

#### Link

---

기존 a태그와 유사하게 페이지 이동을 한다. 하지만 페이지 이동 시 새로고침되는 a 태그와는 다르게 **Link 컴포넌트는 새로고침없이 필요한 컴포넌트만 업데이트**하므로, **빠른 페이지 전환**이 가능하다.🚀

<br />

#### Link 사용방법

---

Link 컴포넌트는 단**순하게 누르면 바로 페이지가 이동**되는 로직에 쓰인다. 예를 들면 네비게이션 메뉴, 혹은 쇼핑몰에 목록을 누르는 것처럼 말이다.

```jsx
<Link to="/path">바로가기</Link>

// or

<Link to="path">
    <div className="contents">
    ...
    </div>
</Link>
```

<br /><br />

#### useNavigate()

---

해당 함수도 url을 전달받아 페이지를 이동할 수 있다. 또한 해당 메소드에 숫자를 넘기면 현재 페이지에서 해당 숫자만큼 히스토리가 이동가능하다. `ex) navigate(-1) : 이전 페이지 이동`

<br />

#### useNavigate() 사용방법

---

**useNavigate()는 동적인 이벤트나 조건에 따라 페이지를 이동할 때** 유용하게 쓰인다. 예를 들면 이메일, 비밀번호 유효성 검사에 통과해야 특정 페이지로 이동과 같은 조건이 따라올 때 쓴다.🙆🏻‍♀️

```jsx
import { useNavigate } from "react-router-dom";

export default function Home {
  const navigate = useNavigate();

  const handleButtonClick = () => {
    navigate("/about");
  };

  return (
    <div>
      <h1>Welcome to Home Page</h1>
      <button onClick={handleButtonClick}>Go to About</button>
    </div>
  );
};
```

<br /><br /><br />

#### 차이점

---

둘 다 페이지 이동을 위해 사용되는데 차이점이 뭐냐?! 한다면 조건이 있냐없냐겠다!
<span id="includes-reduce" style="color: #ff5100;;">Link는 아무 조건 없이 페이지를 이동할때 쓰고, useNavigate()는 함수 내에서 어떠한 조건에 부합하면 페이지 이동!</span>
