---
layout: post
title: 리액트 useParams와 쿼리스트링
subtitle: url에서 데이터 가져오기 대작전
categories: react
tags: [react, memoization]
image: /assets/images/2023-07-07-useParams/useParams.png
---

#### 들어가며😃

---

next.js로 리액트를 입문하다보니 리액트를 공부하면 넥스트가 정말 편하구나 라는 역체감을 자주하게된다. 다시 돌아 리액트를 공부한다는게 어쩌면 귀찮고 어려워 흥미가 떨어질 수 있겠지만, 오히려 나에겐 재밌게 느껴진다. 참 다행인 부분이다. 파도 파도 재밌는 리액트의 세계.. 어쩌면 난 전생에 리액트 써퍼...?🌊🏄🏻🧐 크흠... 어쨋던 이번엔 url에서 정보를 꺼내오는 법에 대해 정리해보려한다. 넥스트에서는 router.query.id 식으로 당연하게 썼었고, 초반 극극극 개린이 시절 다른 기능들을 배우느라 급급해 해당 부분에 대한 원리를 알아볼 여유가 없었다. 그리고 리액트로 돌아가 학습하며 아~ 그래서 그랬구나~ 넥스트에서는 리액트의 이 기능을 이렇게 사용했던 거구나~ 라고 다시 역으로 깨닫는 중이다. 여튼 이번에 useParams, 쿼리스트링 포스팅으로 url에서 정보를 추출하는 방식에 대해 정리해보려 한다!

<br /><br />

### useParams() 이란

---

리액트에서 라우터 사용 시 useParams 훅을 사용하여 url의 동적인 파라미터 정보를 가져올 수 있다. <em style='font-size: 16px; color: #BAD1C2; font-weight: bold;'>동적인 파라미터? 이게 무슨말이냐?🤷🏻‍♀️</em> 하믄 url의 구조에서 `...url/path/1`에서 `1`이 파라미터가 되는 것이다. 예를 들면 쇼핑몰에서 3번 게시글에 들어간다면 url이 www.shopping.com/items/3 이 되고, 7번 게시글에 들어간다면 www.shopping.com/items/7 의 형식이 되는 데이터에 따른 동적인 값이다. 그리고 이후 url에서 3 또는 7이라는 데이터를 뽑아와 api 통신을 하면 된다.

<br /><br />

### useParams() 사용법

---

우선 위에서 언급한 이 동적인 행위를 위해 페이지 라우팅을 설정해주어야한다. 아래를 참고하자.

```jsx
import { BrowserRouter as Router, Route, Routes } from "react-router-dom";
import UseParams from "./components/useParams";

function App() {
  return (
    <>
      <Router>
        <Routes>
          <Route path="/useParams/:id" element={<UseParams />} />
        </Routes>
      </Router>
    </>
  );
}

export default App;
```

위에서 /useParams/:id 페이지 경로를 추가해주었다. 여기서 <span style="color: #ff5100;;">:id는 동적인 파라미터 값을 나타내며</span> 어떤 이름을 갖던 상관없지만, 라우팅 된 컴포넌트에서 파라미터 정보를 받아오려면 경로에서 지정한 이름(id)으로 불러와야 한다. 이 부분을 아래에서 알아보자.

<br />

```jsx
import { useParams } from "react-router-dom";

export default function UseParams() {
  const { id } = useParams();

  return (
    <div>
      <div>현재 파라미터는 {id} 입니다.</div>
    </div>
  );
}
```

useParams()를 사용해 /useParams/:id 의 경로 값으로 설정해주었던 id 값을 불러온다. 그러면 아래와 같이 짜잔! id값에 어떤 값을 넣어주던 페이지가 string 타입의 id 값을 꺼내온다!

<img width="400" alt="파라미터 변경에 따른 화면" src="https://github.com/ju-ju2/precamp_class/assets/71650663/2290c11b-7b35-4016-9e96-85b3b1b4ccb2">

<br />

<em style='font-size: 18px; color: #BAD1C2; font-weight: bold;'>만약 가져온 파라미터로 api 통신을 하고싶다면?!🤷🏻‍♀️</em>  
코리안 제이슨에서 제공해주는 api를 사용해서 통신이 잘 되는지 로그를 찍어보자.

```jsx
import { useParams } from "react-router-dom";

export default function UseParams() {
  const { id } = useParams();
  fetch(`https://koreanjson.com/posts/${id}`)
    .then((response) => response.json())
    .then((json) => console.log(json))
    .catch((error) => console.log(error));

  return (
    <div>
      <div>현재 파라미터는 {id} 입니다.</div>
    </div>
  );
}
```

<img width="500" alt="파라미터 변경에 따른 코리안제이슨 통신 확인" src="https://github.com/ju-ju2/precamp_class/assets/71650663/6665e7ae-7975-4fc0-828d-20859f55272d">
<br />
아주 잘되는 모습이다.😎

<br /><br />

### 쿼리스트링이란

---

<span style="color: #ff5100;;">쿼리스트링은 url의 한 부분으로써 특정 매개변수와 값을 전달하는데 사용</span>된다. 예를 들어`color="red"`라는 정보를 마치 컴포넌트가 props를 받듯, 데이터를 url로 전달하는 것이다.

쿼리스트링은 `www.example.com/value?title=제목&content=내용` 과 같은 형식을 띄는데, url 끝의 ? 부터 시작하고 key=value 형식으로 매개변수와 값을 지정한다. 갯수는 상관없다. 아래에서 사용법을 자세히 알아보자.

<br /><br />

### 쿼리스트링 사용법

---

우선 쿼리스트링을 사용할 컴포넌트에 페이지 경로를 지정해주자. 그리고 이번에는 가상의 데이터를 뿌려주고, 해당 데이터 클릭 시 페이지가 넘어가는 로직을 구현해 볼 것이다. 그래서 `/queryString` 과 `/queryString/value` 두가지 페이지 경로를 지정해주었다.

```jsx
import { BrowserRouter as Router, Route, Routes } from "react-router-dom";
import QueryStringPage from "./pages/queryString";
import QueryString from "./components/queryString";

function App() {
  return (
    <>
      <Router>
        <Routes>
          <Route path="/queryString" element={<QueryStringPage />} />
          <Route path="/queryString/value" element={<QueryString />} />
        </Routes>
      </Router>
    </>
  );
}

export default App;
```

<br />
아래에서 데이터를 만들어주었고, map() 메소드를 이용하여 각 데이터에 onClick 이벤트를 할당하고, useNavigate 훅을 사용해 페이지를 라우팅한다. 그리고 전달할 값들을 url에 할당 시키면 데이터를 넘겨주는 것이다.

```jsx
import { useNavigate } from "react-router-dom";
export default function QueryStringPage() {
  const data = [
    { id: 1, title: "제목1", content: "내용1" },
    { id: 2, title: "제목2", content: "내용2" },
    { id: 3, title: "제목3", content: "내용3" },
  ];
  const navigate = useNavigate();
  const onClickBtn = (data) => () => {
    navigate(`/queryString/value?title=${data.title}&content=${data.content}`);
  };
  return (
    <div>
      <h3>쿼리스트링</h3>
      <div>
        {data.map((data) => (
          <div key={data.id} onClick={onClickBtn(data)}>
            <p>{data.title}</p>
            <p>{data.content}</p>
          </div>
        ))}
      </div>
    </div>
  );
}
```

<img width="300" alt="쿼리스트링 화면" src="https://github.com/ju-ju2/precamp_class/assets/71650663/34697505-5d01-43e4-af00-490bc897d40b">

<br />
그리고 클릭이벤트로 라우팅 된 컴포넌트에서 url로 부터 값을 받아오려면 아래와 같은 절차가 필요하다.

```jsx
import { useLocation } from "react-router-dom";

export default function QueryString() {
  const location = useLocation();
  const queryParams = new URLSearchParams(location.search);
  const title = queryParams.get("title");
  const content = queryParams.get("content");
  return (
    <div>
      <div>{title}</div>
      <div>{content}</div>
    </div>
  );
}
```

1. location.search : useLocation 훅을 사용하여 현재 url 쿼리스트링 가져오기.(?title=ooo&content=ooo 와 같은 꼴)
2. URLSearchParams : location.search의 쿼리스트링을 파싱하고 변수에 할당.
3. queryParams : 매개변수와 값 추출(queryParams.get("title"))
4. (선택)쿼리 숨기기 : `window.history.replaceState({}, null, location.pathname)` 구문을 추가하면 쿼리를 숨길 수 있다.

<br /> 
위와 같은 과정을 거치면 아래와 같이 url에서 데이터를 받아올 수 있다. 끝!

<img width="400" alt="쿼리스트링 라우팅 된 화면" src="https://github.com/ju-ju2/precamp_class/assets/71650663/c4ebd526-f0dc-4111-9ac2-8ab69af5a200">

<br /><br /><br /><br />

---

오늘도 유익한 시간이었다. 사실 넥스트를 쓸때도 쿼리 파라미터를 넘기는 건 안해봤어서 한번 찾아보니 객체로 넘겨주는 방식으로 개념이 비슷한 것 같다. 다음에는 넥스트에서 쓰이는 방식도 함께 기록해봐야겠다. 오늘은 마무리! 오늘 성장 완💪🏻
