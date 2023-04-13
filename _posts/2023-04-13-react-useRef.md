---
layout: post
title: 리액트 useRef
subtitle: 목표는 너다!
categories: react
tags: [react]
image: /assets/images/2023-04-13-useRef/useRef.png
---

자바스크립트에서는 DOM 요소에 class나 id를 주고, document.getElementById(), document.querySelector() 등 과 같은 함수를 사용해 DOM을 직접 조작하였다. 하지만 리액트에서는 DOM을 직접적으로 조작하는 것을 권장하지 않는다. 그런데 프로젝트를 진행하다보면 특정한 DOM요소를 콕 찝어 조작해야할 때가 생긴다. 그렇다면 `useRef`를 사용해보자.
<br/>
<br/>
<br/>

### useRef

---

우선 ref, 즉
<span style=' font-weight: bold'>reference로 참조</span>를 뜻한다.  
useRef는 React에서 제공하는 hook 중 하나로, React 함수형 컴포넌트에서 Ref를 사용할 수 있게 한다. 그리고 `.current `프로퍼티를 통해 실제 노드나 인스턴스를 참조할 수 있게 해준다.

<br/>

간단한 예제를 들어보자. 네모박스 하나와 버튼창이 있다.

<div style="display: flex; flex-direction: column; align-items: center; width: 300px; height: 300px;" >
<div style="width: 300px; height: 200px; background-color: skyblue; margin-bottom: 10px"></div>
<button style="width: 100px;">색상변경</button>
</div>
  
나는 버튼을 누를때 네모박스의 색깔을 변경하고 싶다. 로직은 간단하다.

```jsx
import { useRef } from "react";

function Example() {
  // 1. 변경하고 싶은 네모박스에 ref 참조를 생성한다.
  const boxRef = useRef(null);

  // 2. 버튼 클릭 참조 된 박스의 색깔을 변경한다.
  const changeColor = () => {
    boxRef.current.style.backgroundColor = "red";
  };

  return (
    <div>
      <div ref={boxRef} style={{ backgroundColor: "ivory" }}></div>
      <button onClick={changeColor}>색상변경</button>
    </div>
  );
}
```

이렇게 하면 버튼을 누를 때, 참조된 태그를 찾아 그 속성을 변경할 수 있다.  
물론 useRef가 실제로는 더 복잡한 로직에 쓰이고 더 많은 기능을 하지만, 위의 예제에서는 useRef에 대한 개념을 이해하기 쉽게 접근해보았다.

<br/>
<br/>
<br/>

또한 setState 처럼 useRef를 사용하여 **컴포넌트의 변수도 생성**할 수 있다.

---

```jsx
import { useRef } from "react";

function MyComponent() {
  const countRef = useRef(0);

  const handleClick = () => {
    countRef.current += 1;
    console.log(`Button clicked ${countRef.current} times`);
  };

  return (
    <div>
      <button onClick={handleClick}>Click me</button>
    </div>
  );
}
```

위 코드에서의 로직은 간단하다. useRef를 사용하여 countRef를 생성하고, 0으로 초기화한다. handleClick 함수에서 countRef.current를 사용하여 countRef 변수를 업데이트하고, 업데이트된 값을 콘솔에 출력한다.

사실 위의 예제만 보면 `useState`함수와 별반 다를게 없어보인다. 하지만 useState와는 달리 **useRef는 상태가 업데이트 되어도 리랜더링 되지 않는다!**
<br/>
<br/>
<br/>
<br/>
<br/>
이러한 특성으로 **컴포넌트의 상태를 유지**하고 싶을 때도 useRef가 사용될 수 있다.
<em style='font-size: 20px; color: #BAD1C2; font-weight: bold;'>그럼 useRef를 사용해서 컴포넌트 상태를 유지해야할 때는 언제일까?🤷🏻‍♀️</em>
<br/>
<br/>

**1. 컴포넌트의 상태가 변경되더라도 다시 렌더링하지 않아야 하는 경우**

일부 컴포넌트의 상태는 변경되더라도 다시 렌더링 할 필요가 없는 경우가 있다. 타이머나 애니메이션의 상태같은거 말이다. 화면에서 랜더링 되는 요소가 아닌 내부에서만 처리되는 것은 useRef를 사용하여 상태를 유지할 수 있다.

**2. 이전 상태를 참조해야 하는 경우**

특정 상태가 변경되었을 때, 이전 상태와 비교하여 특정 동작을 수행해야 하는 경우에 useRef를 사용하여 이전 상태를 유지할 수 있다.
<br/>
<br/>
<br/>
