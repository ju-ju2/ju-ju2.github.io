---
layout: post
title: 리액트 함수형 컴포넌트와 클래스형 컴포넌트 차이
subtitle: 클래스형 컴포넌트를 함수형 컴포넌트로 바꾸자
categories: react
tags: [react]
image: /assets/images/2023-04-17-component/component.png
---

나는 리액트를 올해 배우기 시작했기 때문에 처음부터 함수형 컴포넌트로 프로젝트를 꾸려나갔다. 그런데 모르는 것이 있어 구글링을 하다보면 **함수형 컴포넌트말고 익숙하지 않는 클래스형 컴포넌트**가 예제로 주어질 때가 있다.
그럴 때 마다 당혹스럽기도하고 키워드를 바꿔서 찾기도 했다. 현재는 클래스형 컴포넌트가 사용되지 않는 추세라고 해서 배울필요가 있을까? 하지만,,,  
내가 어떤 회사에 취업해서 어떤 업무를 할 지 모르고, 그 회사가 여전히 클래스형 컴포넌트를 사용할 수 있고, 내 업무가 기존 클래스형 컴포넌트를 함수형 컴포넌트로 바꾸는 업무를 할 수도 있는 것이다.
이 외에도 내가 클래스형 컴포넌트를 마주칠 수 있는 상황이 많기 때문에 앞으로 사용하지는 않더라도 읽을 줄 알아야하며 바꿀 줄 알아야한다. 그래서 써보는 오늘의 포스팅.
<br/>
<br/>
<em style='font-size: 20px; color: #BAD1C2; font-weight: bold;'>클래스형 컴포넌트와 함수형 컴포넌트의 차이를 알고 전환해보자!🙆🏻‍♀️</em>
<br/>
<br/>

### 1.선언방식

---

<div style='color: #454545; font-weight: bold;'>클래스형 컴포넌트</div>
클래스형 컴포넌트는 class 라는 키워드를 이용하여 선언한다. 클래스 이름은 대문자로 시작해야 되며 React.component 클래스를 상속받아 구현한다.
extends 라는 키워드 처럼, 클래스를 확장하여 Component의 기능을 상속받는 것이다. 여기서 `render()`메소드를 통해 화면을 그려준다.

```jsx
import React, { Component } from "react";

class MyComponent extends Component {
  render() {
    return (
      <div>
        <h1>hello!!</h1>
      </div>
    );
  }
}

export default MyComponent;
```

<br/>

<div style='color: #454545; font-weight: bold;'>함수형 컴포넌트</div>
함수형 컴포넌트는 이름 그래도 함수를 이용하여 선언한다. 함수 이름은 역시 대문자로 시작해야한다.

```jsx
import React from "react";

function MyComponent() {
  return (
    <div>
      <h1>hello!!!</h1>
    </div>
  );
}

export default MyComponent;
```

<br/>
<br/>

### 2.props

---

<div style='color: #454545; font-weight: bold;'>클래스형 컴포넌트</div>
`this.props`를 사용하여 props를 참조한다.

```jsx
import React, { Component } from "react";

class MyComponent extends Component {
  render() {
    return (
      <div>
        <h1>{this.props.title}</h1>
        <p>{this.props.text}</p>
      </div>
    );
  }
}

export default MyComponent;
```

<br/>

<div style='color: #454545; font-weight: bold;'>함수형 컴포넌트</div>
함수의 인자로 props를 받을 수 있다.

```jsx
import React from "react";

function MyComponent(props) {
  return (
    <div>
      <h1>{props.title}</h1>
      <p>{props.text}</p>
    </div>
  );
}

export default MyComponent;
```

<br/>
<br/>

### 3.이벤트핸들링

---

<div style='color: #454545; font-weight: bold;'>클래스형 컴포넌트</div>
클래스형 컴포넌트에서 이벤트 핸들링을 구현할 때는 this를 사용하여 메서드를 정의한다.

```jsx
class MyComponent extends React.Component {
  handleClick() {
    // ...
  }

  render() {
    return <button onClick={this.handleClick}>Click me!</button>;
  }
}
```

<br/>
<div style='color: #454545; font-weight: bold;'>함수형 컴포넌트</div>
this를 바인딩 할 필요가 없다.이벤트 핸들러를 선언할 때 const를 사용하면 함수를 재사용할 수 있는 장점이 있기 때문에 const를 사용하는 것이 좋다.

```jsx
function MyComponent() {
  function handleClick() {
    // ...
  }

  return <button onClick={handleClick}>Click me!</button>;
}
```

<br/>
<br/>

### 4.state

---

<div style='color: #454545; font-weight: bold;'>클래스형 컴포넌트</div>
클래스형 컴포넌트에서도 역시 setState 메소드가 있다. 그런데 사용하는 방식이 함수형 컴포넌트보다 약간 복잡하다. `constructor(props)` 메소드 내에서 `super(props)`를 호출하여 부모 클래스의 생성자를 호출하고, `this.state` 객체를 사용하여 컴포넌트의 초기 상태를 설정한다. 이렇게 초기 상태를 설정하고, 이후에 setState() 메소드를 사용하여 상태를 변경한다.

```jsx
import React, { Component } from "react";

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0,
    };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.handleClick}>Increment</button>
      </div>
    );
  }
}

export default Counter;
```

위의 로직은 이벤트 헨들러와도 관련 있다. 메서드를 이벤트 핸들러로 사용하려면, 이벤트 핸들러에서 this가 클래스 인스턴스를 가리키도록 바인딩해주어야 하기 때문에 `this.handleClick = this.handleClick.bind(this);`구문이 추가 되었다.

너무 복잡하다. 하지만 constructor 메소드를 사용하지 않고 state를 변경하는 방법이 있다. `클래스 필드 선언 방식`(class fields syntax)을 사용하는 것이다.
예를 들어, 클래스 내부에서 state 객체를 정의할 때 constructor 메소드를 사용하는 대신에 아래와 같이 state = { key: value } 형태로 선언할 수 있다.  
이 방법은 코드가 간결해지고, constructor 메소드를 생략할 수 있어서 코드 가독성이 좋아진다. 하지만 IE11을 포함한 구형 브라우저에서는 지원되지 않기 때문에 주의가 필요하다고 한다.

```jsx
class Counter extends React.Component {
  state = {
    count: 0,
  };

  // 이하 동일...
}
```

<br/>

<div style='color: #454545; font-weight: bold;'>함수형 컴포넌트</div>
함수형 컴포넌트에서는 `useState` hook을 통해 상태를 쉽게 업데이트 할 수 있다. 아래의 예제를 보면 count라는 변수가 있고, 초기값은 useState()에 설정된다. setCount()는 초기값을 변경하는 메소드이다.

```jsx
import React, { useState } from "react";

function MyComponent(props) {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  // ...
}
```

### 5.컴포넌트 생명주기

---

우선 리액트에서 컴포넌트가 처음 화면에 나타날 때를 '마운트'라고 하고, 화면에서 사라질 때를 '언마운트'라고 한다.

<div style='color: #454545; font-weight: bold;'>클래스형 컴포넌트</div>
클래스형 컴포넌트에서는 componentDidMount, componentDidUpdate, componentWillUnmount와 같은 라이프사이클 메소드가 있는데 각각 컴포넌트가 마운트되었을 때, 업데이트될 때, 언마운트될 때 실행된다.

```jsx
import Router from "next/router";
import { Component } from "react";

export default class MyComponent extends Component {
  state = {
    count: 0,
  };

  componentDidMount() {
    console.log("컴포넌트가 마운트됐습니다~");
  }

  componentDidUpdate() {
    console.log("컴포넌트가 변경됐습니다~");
  }

  componentWillUnmount() {
    alert("컴포넌트가 제거됩니다~");
  }

  onClickButton = () => {
    this.setState((prev) => ({ count: prev.count + 1 }));
  };

  onClickMove = () => {
    Router.push("/");
  };

  render() {
    console.log("마운트 시작");
    return (
      <>
        <div>카운트: {this.state.count}</div>
        <button onClick={this.onClickCounter}>카운트(+1)</button>
        <button onClick={this.onClickMove}>이동하기</button>
      </>
    );
  }
}
```

<br/>

<div style='color: #454545; font-weight: bold;'>함수형 컴포넌트</div>
함수형 컴포넌트에서는 이를 모드 `useEffect`로 대체할 수 있다. useEffect의 두 번째 인자로 `의존성 배열`(dependency array)을 전달하며 useEffect가 특정한 조건에만 실행되도록 제어한다.  
의존성 배열은 배열 안에 들어있는 값이 변화할 때만 useEffect를 실행시키도록 하는 역할을 한다. 만약 의존성 배열이 비어있다면, useEffect는 컴포넌트가 처음 렌더링될 때 한 번만 실행된다.

```jsx
import { useRouter } from "next/router";
import { useEffect, useState } from "react";

export default function MyComponent() {
  const [count, setCount] = useState(0);
  const router = useRouter();

  useEffect(() => {
    console.log("처음에만 실행되는 로그 / componentDidMount()");
    return () => {
      console.log("종료할때만 실행되는 로그 / componentWillUnmount()");
    };
  }, []);

  useEffect(() => {
    console.log("변경될때마다 실행되는 로그 / componentDidUpdate()");
  });

  const onClickButton = () => {
    setCount((prev) => prev + 1);
  };

  const onClickMove = () => {
    router.push("/");
  };

  return (
    <>
      <div>카운트: {count}</div>
      <button onClick={onClickButton}>카운트(+1)</button>
      <button onClick={onClickMove}>이동하기</button>
    </>
  );
}
```
