---
layout: post
title: 리액트 렌더링 최적화 - 메모이제이션
subtitle: 무턱대고 쓰면 아니아니 아니되오
categories: react
tags: [react, memoization]
image: /assets/images/2023-07-04-immutable/memoization.png
---

#### 들어가며😃

---

context API의 개념을 접하다가 React.memo를 사용해보게 되었고 따로 더 공부하고 싶었던 memoization 부분을 참 타이밍에 잘맞게 프리온보딩 시간에 배우게 되었다. 암 어 럭키 걸😎 그리고 해당 개념을 적용하기 전에는 더 많은 핵심 개념들이 탄탄히 잡혀야 한다는 것을 깨닫게 되었고 자바스크립트 엔진의 메모리 구조와 할당 부분을 먼저 정리해보았다. 그리고 이번 포스팅에서는 `리액트 렌더링 최적화`와 `React.memo`, `useMemo` 에 대해 정리해보려한다.

<br /><br />

### 렌더링이란?

---

리액트에서 렌더링이란 화면에 특정한 요소를 그려내는 것을 의미한다. 👩🏻‍🎨 그리고 이 <span style="color: #ff5100;;">렌더링 과정을 잘 처리해주는 것이 리액트와 같은 프레임워크를 사용하는 이유</span>이다.  
브라우저는 화면을 보여주기 위해서 HTML, CSS, JavaScript를 다운로드 받고, DOM을 계산하고 CSSOM을 결합시키고, 위치를 계산하여 최종적으로 화면에 픽셀 형태로 그리는데, 이 과정을 `CRP(Critical Rendering Path)`라고 부른다. 그런데 Vanilla Javascript 에서는 하나하나의 DOM에 직접 접근하고 수정하는데(`document.getElementById.~~`) 이는 브라우저에게 많은 연산을 요구하게 되고, 퍼포먼스를 저하시키는 요인이 될 수 있다. 리액트는 이를 해결하고자 **VirtualDOM**이란 개념을 도입한 것이다. 그래서 리액트는 보여주고 싶은 핵심 UI를 “선언”하기만 하면 실제로 DOM을 조작해서 UI를 그려내고, 변화시키는 일은 라이브러리나 프레임워크가 대신 해주는 방식을 찾게 된다. **최종적으로 리액트를 사용하면 개발자는 UI를 설계하는대에만 집중**할 수 있게 되는것이다.

<br /><br />

### 리렌더링이 되는 시점

---

리액트에서는 데이터가 변경되고 UI 가 변경된 데이터를 반영해 변할 수 있도록 state를 변경시키는 방법을 setState로 한정시킨다. 그리고 이 함수가 호출될 때 마다 리렌더링되도록 설계되어있다. 이러한 이유로 <span style="color: #ff5100;;">리액트에서 렌더링이 발생하는 시점은 state의 상태가 변할 때</span>이다. 그런데 특정 컴포넌트의 state 가 변하면 해당 컴포넌트와 그 자식 컴포넌트들이 모두 리렌더링 되는 현상이 발생한다. 이러한 **리액트의 멘탈 모델을 잘 이해하고 있는 것이 리액트를 이용해서 프로젝트를 설계하고, 최적화하는데 가장 기본**이 될 것이다!

<br /><br />

### React.memo

---

위에서 state가 변하면 해당 컴포넌트에 속한 자식 컴포넌트 모두가 리렌더링 된다고 하였다. 그런데 말이다,,, 하위 컴포넌트가 받는 props를 받지않는 컴포넌트가 있을 수도 있고, props를 받아도 state의 변화가 없어 랜더링을 다시 해줄 필요없는 컴포넌트도 있을 것이다. 이런 경우에는 굳이 하위 컴포넌트 함수를 호출할 필요 없이, 이전에 저장되어있던 결과를 그대로 사용하는 것이 효율적일 것이다.

<em style='font-size: 18px; color: #BAD1C2; font-weight: bold;'>그럼 처음부터 변화를 감지하도록 설계하면 안됐나?🤷🏻‍♀️</em>

하지만 변화가 있는지 없는지 매번 렌더링 과정에서 하나하나 컴포넌트 트리를 순회하면서 검사하는 것은 굉장히 비효율적일 것이다. 그래서 리액트는 개발자에서 이 컴포넌트가 리렌더링 되어야할지 아닐지에 대한 여부를 표현할 수 있는 `React.memo`라는 함수를 제공하고 기존의 컴포넌트 UI를 재사용할지 말지 판단하는 권한을 주었다.

<br /><br />
<span style="color: #ff5100;;">React.memo는 컴포넌트의 리렌더링을 최적화하기 위해 사용</span>되는 것이다. 고차 컴포넌트(`HOC  : Higher-Order Component`)로 컴포넌트가 동일한 props를 받았을 때, 이전에 렌더링된 결과와 비교하여 변화가 없다면 UI를 재사용하여 불필요한 리렌더링을 방지할 수 있는 것이다.

React.memo를 쓰기 위해선 메모이제이션 하고 싶은 컴포넌트를 React.memo로 감싸주면 된다. 간단히 아래와 같은 형식이다.

```javascript
import React from "react";

const MyComponent = React.memo((props) => {
  // 컴포넌트의 내용
});

export default MyComponent;
```

<br />

위를 참조하여 아래의 예시를 보자. text라는 state를 props로 받는 컴포넌트가 memoization 처리가 된 경우, 안된 경우, 두번째 파라미터의 AreEqual 함수에 false가 return된 경우, true가 return 된 경우 이다.  
text는 state를 넘겨주기 위해 설정한 값이고, rerender 함수는 state값은 넘겨주지는 않지만 리렌더링을 일으키기 위해 넣어준 함수이다.

```javascript
import React, { useState } from "react";

export default function Memo() {
  const [text, setText] = useState("");
  const [_, setState] = useState(0);
  const rerender = () => setState((prev) => prev + 1);
  return (
    <div>
      <div>memoization Test</div>
      <div>
        <input value={text} onChange={(e) => setText(e.currentTarget.value)} />
      </div>
      <button onClick={rerender}>re-render</button>
      <ChildComponent name="memo X" value={text} />
      <MemoizedComponent name="memo O" value={text} />
      <ReturnFalseMemo name="return false" value={text} />
      <ReturnTrueMemo name="return true" value={text} />
    </div>
  );
}

const ChildComponent = ({ name, value }) => {
  console.log(`${name} page re-rendered`);
  return (
    <div>
      {name} : {value}
    </div>
  );
};

const MemoizedComponent = React.memo(ChildComponent);
// 이전과 새로운 props를 얕은 비교를 통해 변경된 부분이 있다면 리렌더링하겠다.
const ReturnFalseMemo = React.memo(ChildComponent, () => false);
// props는 서로 다르다. 리렌더링 하겠다.
const ReturnTrueMemo = React.memo(ChildComponent, () => true);
// props는 서로 같다. 리렌더링 하지 않겠다.
```

React.memo가 씌어진 부분을 보면 기본적으로 이전 컴포넌트와 변화된 컴포넌트의 **얕은 비교**가 진행된다. 만약 특정히 변경을 감지하고 싶은 값이 있다면, 두번째 파라미터에 true, false 를 반환하는 propsAreEqual 함수를 넣어 memo 기능을 활성화 시킬 수 있다. 간단히 아래와 같은 로직이다.

```javascript
function ChildComponent(props) {}

function areEqual(prevProps, nextProps) {
  /*
  React.memo의 두번째 인자에서 사용되는 함수는 리액트가 알아서 매개변수에 prevProps 와 nextProps를 넣어준다.
  */
  return prevProps === nextProps;
  // **얕은 비교
  // true를 return할 경우 이전 결과를 재사용
  // false를 return할 경우 리렌더링을 수행
}

export default React.memo(ChildComponent, areEqual);
```

<img width="300" alt="React.memo 예제" src="https://github.com/ju-ju2/precamp_class/assets/71650663/f27b05af-cea6-41bc-8966-5c068cf62c7c">

<img width="300" alt="React.memo 예제" src="https://github.com/ju-ju2/precamp_class/assets/71650663/8fd57c92-94fa-4e08-8830-373bb056c5c8">

예제를 실행시키면 위과 같은 결과가 나온다.  
이미지를 보면 가장 아래 areEqual함수에서 true를 반환하는 컴포넌트는 UI를 무조건 적으로 재사용한다는 것으로 props가 변경되어도 리렌더딩 되지 않는 않는 모습을 볼 수 있다.  
그리고 리랜더링 버튼을 눌러 부모 컴포넌트가 리렌더링 되어도 memoization 된 컴포넌트는 리렌더링 되지 않는 모습을 확인할 수 있다.

<br/><br/>

### ⚠️ memo 사용 시, 주의할점

---

memo는 props의 얕은 비교만 수행하기 때문에, 객체나 배열과 같은 참조 타입의 props가 변경되더라도 그 내부의 값이 동일하다면 리렌더링을 방지할 수 없다. 이 개념을 이해하기 위해 이전에 자바스크립트의 메모리 구조를 이해하는 시간을 가진 것이다. 어쨌던 이 경우에는 내부의 값이 변경되었는지를 확인하는 로직을 추가해주어야 한다. 아래를 참고해보자.

```javascript
const areEqual = (prevProps, nextProps) => {
  if (prevProps.name !== nextProps.name) return false;
  if (prevProps.hello !== nextProps.hello) return false;
  if (prevProps.object !== nextProps.object) return false;
  if (prevProps.array !== nextProps.array) return false;

  return true;
};
```

props가 참조타입으로 표현된다면, 렌더링될 때 마다 새롭게 생성되기때문에 `prevProps === nextProps` 와 같은 얕은 비교로 비교하는 것은 의미가 없다. 당연히 매번 다르다고 인식하기 때문에 매번 새롭게 리랜더링 될 것이다. 그렇기 때문에 위와 같이 **props 객체 안의 각 property들을 비교해서 state가 변경되었는지를 판단**해줘야하는 것이다.

<br /><br />

### useMemo

---

<em style='font-size: 18px; color: #BAD1C2; font-weight: bold;'>위에서 만약 props에 함수를 전달해주면 어떻게 될까?🤷🏻‍♀️</em>  
함수는 기본적으로 이전 호출과 새로운 호출간에 값을 공유할 수 없다. 그래서 함수를 props로 전달하면 memo 처리를 해도 계속 리렌더링 된다. 아래를 참고해보자.

```javascript
import React, { useState } from "react";

export default function Immutable() {
  console.log("parent-re-rendered");

  const [text, setText] = useState("");
  const [_, setRender] = useState(0);
  const makeRerender = () => {
    setRender((prev) => prev + 1);
  };
  const multiplyCount = () => {
    return { num } * 2;
  };
  return (
    <div>
      <div>참조타입 리렌더링 테스트</div>
      <div>
        <input value={text} onChange={(e) => setText(e.currentTarget.value)} />
      </div>
      <div>
        <button onClick={makeRerender}>re-render</button>
      </div>
      <MemoizedChild value={text} expression={multiplyCount} />
    </div>
  );
}

const ChildComponent = ({ value }) => {
  console.log("child-re-rendered");
  return <div>text : {value}</div>;
};

const MemoizedChild = React.memo(ChildComponent);
```

<img width="300" alt="props 함수 전달" src="https://github.com/ju-ju2/precamp_class/assets/71650663/5fd2e8c3-e5e6-4b90-9fcf-02f2d6bc4ca6">

위에서 보면 ChildComponent를 memo로 감싸주었지만 계속 리렌더링되는 것을 볼 수 있다.

만약 특정한 함수 호출 내에서 만들어진 변수를 다음 함수 호출에도 사용하고 싶다면 그 값을 외부에 저장해두었다가 다음 호출때 다시 꺼내와야한다. 약간 번거로운 일인데 리액트는 함수 컴포넌트에서 `값`을 memoization 할 수 있도록 useMemo라는 기능을 제공하고 있다.

위에서 나타나는 문제를 useMemo를 통하여 해결해보자.

```javascript
const multiplyCount = useMemo(() => ({ num } * 2), [num]);
```

<img width="300" alt="props 함수 전달" src="https://github.com/ju-ju2/precamp_class/assets/71650663/da3a638a-122d-4572-a70e-23f07a1772af">

useMemo를 통해 값을 memo해줬더니 렌더링이 되고 있지않다. 위에서 보듯이 useMemo는 두가지 인자를 받는다. **첫번째 인자는 콜백함수**이며 이 함수에서 리턴하는 값이 메모된다. **두번째는 의존성 배열**을 받게 되고 해당 배열에 있는 값 중 하나라도 이전 렌더링과 비교하여 변경사항이 있다면 새로운 값을 다시 계산한다.

<br /><br />

### useCallback

---

useMemo가 값을 메모이제이션 해주었다면, <span style="color: #ff5100;;">useCallback은 함수 자체를 메모이제이션하여 재사용하는데 사용</span>된다. 일반적인 값들은 useMemo를 통해서 메모하기 편리하다. 하지만 함수의 경우 useMemo를 사용해서 함수를 메모하게되면 콜백함수에서 또 다른 함수를 리턴하는 꼴이 된다. 아래와 같은 형식 말이다.

```javascript
const memorizedFunction = useMemo(() => () => alert("Hello World"), []);
```

위는 동작 상에 아무런 이상이 없지만 문법적으로 다소 불편해질 수 있다. 그래서 useCallback은 이러한 동작을 간소화 시킨 것이며, 아래와 같다.

```javascript
const memorizedFunction = useCallback(() => alert("Hello World"), []);
```

<br /><br />

### 언제 최적화를 진행할까?

---

음,,메모이제이션이라는 개념을 위에서 정리해봤을때는, 안 쓸 이유가 없다. 굉장히 효율적이고 사용하기만 하면 최적화가 무조건 될 것 같은 느낌적인 느낌이 든다. 하지만 실상은 명확한 목적없이 무조건적인 메모이제이션을 사용하는 것은 비효율적이라고 한다. 우선 메모이제이션은 특정한 값을 저장해두고, 비교하는 것이다. 그렇기 때문에 **새로운 값을 만드는 것과 어딘가에 값을 저장해두고 의존성을 비교해서 값을 대체할지 말지에 어느것이 비용이 더 적게들지 생각**해봐야한다.  
상황에 따라 어떤것이 적절할지는 다르겠지만 새로운 값을 만드는 과정이 복잡하지 않다면 메모이제이션을 사용하는 것은 오히려 비용이 더 많이 들수도 있다. 컴퓨터 자원 측면 뿐만 아니라 메모이제이션을 쓰면 코드의 복잡도가 올라가는 것이기 때문에 개발적인 측면의 비용도 무시할 수 없게된다.  
그렇게 때문에 <span style="color: #ff5100;;">메모이제이션의 필요성을 분석하고 필요하다고 판단되는 순간에만 사용</span>해줘야한다. 즉, 현재의 프로젝트에 성능적인 이슈가 발생했거나, 발생할 가능성이 있고 이를 해결해야 될 필요성이 있는 상황에서 수행하는 것이 최적화인 것이다.

<br /><br /><br /><br />

---

이번 시간에 배웠던 것중 인상깊었던 것은 **최적화는 꽁짜가 아니다**라는 것이다. 최적화를 위해서는 코드가 추가되어야하고 복잡도를 증가시키고, 최적화를 위한 개발자의 시간과 노력이 투자된다. 이는 꽤나 비싼 비용이기 때문에 현업에서는 자기의 도전정신, 탐구욕을 채우기위한 최적화의 시도는 지양하는 것이 좋다고 한다. 물론 사이드 프로젝트에서는 얼마든지 써도 상관없는 얘기..! 여튼 메모이제이션 파트를 공부해보면서 많은 깨달음이 스쳐갔다. 아, 이걸 몰랐다고? 아, memo 무조건 좋네! 아, 무조건 좋은게 아니잖아..? 등.. 아직 전체적으로 아직 배울 것들이 많다는걸 느꼈다는 뜻,,,😰 그래도 중요한 개념들 짚었고, 앞으로의 리액트 사용에 발판이 되어 줄 매우 뿌듯한 포스팅이었다.👩🏻‍💻
