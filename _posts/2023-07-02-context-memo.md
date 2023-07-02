---
layout: post
title: 리액트 context API 메모이제이션
subtitle: 리렌더링 멈춰!
categories: react
tags: [react, hook]
image: /assets/images/2023-07-02-context-memo/context-memo.png
---

#### 들어가며😃

---

이전 포스팅에서 context API 와 reducer를 사용해서 useState 를 대체하는 상태관리 기법을 알아보았다. 그런데 **Provider에서 제공한 value가 달라진다면 useContext를 사용하고 있는 모든 컴포넌트가 리렌더링되는 문제**가 있다는 것을 알게되었다. 해당 이슈를 다루기 위해서는 _메모이제이션_ 과정이 필요했는데, 오늘은 그 과정에 대해 포스팅 해보려한다. 리렌더링 되는 과정은 context API 와 useReducer 훅을 함께 사용해 상태 값을 변경해보며 알아보자.

<br /><br />

### context API 선언

---

```jsx
export const Context = createContext();
```

자 우선 나는 변수를 공유할 것이다!! 라고 선언해주자. 그리고 전달해줄 `value` 를 아래 `useReducer`를 사용하여 상태값을 설정해주자.

### useReducer 상태 설정 및 Provider로 감싸기

---

```jsx
export default function Parent() {
  // 사용자가 지정할 상태 값
  const [number, setNumber] = useState(0);

  const up = () => {
    countDispatch({ type: "up", number: number });
  };
  const down = () => {
    countDispatch({ type: "down", number: number });
  };
  const reset = () => {
    countDispatch({ type: "reset", number: number });
  };

  const countReducer = (presentCount, action) => {
    // 버튼을 누르면 지정된 number 만큼 연산된다.
    if (action.type === "up") {
      return presentCount + action.number;
    } else if (action.type === "down") {
      return presentCount - action.number;
    } else return 0;
  };

  const [count, countDispatch] = useReducer(countReducer, 0);

  const onChangeNum = (e) => {
    setNumber(Number(e.currentTarget.value));
  };
  return (
    <>
      <div>
        <div>{count}</div>
        <button onClick={up}>+</button>
        <button onClick={down}>-</button>
        <button onClick={reset}>reset</button>
        <input
          placeholder="원하는 값"
          type="text"
          value={number}
          onChange={onChangeNum}
        />
      </div>
      <Context.Provider value={count}>
        <First />
      </Context.Provider>
    </>
  );
}

// child 컴포넌트

const First = () => {
  console.log("첫번째");
  return (
    <div>
      <div>첫번째 컴포넌트</div>
      <Second />
    </div>
  );
};

const Second = () => {
  console.log("두번째");
  return (
    <div>
      <div>두번째 컴포넌트</div>
      <Third />
    </div>
  );
};

const Third = () => {
  const count = useContext(Context);
  console.log("세번째");
  return (
    <div>
      <div>세번째 컴포넌트</div>
      {count}
      <Fourth />
    </div>
  );
};

const Fourth = () => {
  console.log("네번째");
  return (
    <div>
      <div>네번째 컴포넌트</div>
    </div>
  );
};
```

위의 코드를 간단하게 설명하자면 `useReducer` 훅으로 **값의 초기값, 액션(dispatch), 액션값에 따른 처리(reducer)를 설정**해준다. 위와 같은 경우에는 사용자가 number 상태를 지정하고 그 +,-,reset 버튼을 누르면 설정된 number 만큼 연산이 되어 count 라는 변수에 할당된다.

그리고 계산된 count 값은 4개의 자식 컴포넌트 중 Third 컴포넌트에 할당된다. 그러면 아래와 같은 상황이 되는데, 여기서 문제도 함께 보자.

<br /><br />
<img width="400" alt="리렌더링 문제" src="https://github.com/ju-ju2/precamp_class/assets/71650663/f90a9b1b-77d7-4ef1-933a-bf590e760a11">

10만큼 연산이 실행되고 세번째 컴포넌트에서 연산된 값을 받는다. 그런데 말이다...  
오른쪽 로그를 살펴보면 첫,둘,셋,넷 컴포넌트가 모두 리랜더링 되고있다.🤢🤢🤢
<span style="color: #ff5100;;">이게 바로 Context API 를 사용할 때 주의할 점이다.</span> 이 문제를 해결하기 위해 **메모이제이션** 과정이 필요하다. 간단히 `React.memo` 를 사용해서 최적화를 시켜보자.

### React.memo를 통한 최적화

---

```jsx
const First = React.memo(() => {
  console.log("첫번째");
  return (
    <div>
      <div>첫번째 컴포넌트</div>
      <Second />
    </div>
  );
});
```

과정은 간단하다. 부모 컴포넌트에서 Provider 로 감싸주었던 First 컴포넌트에 React.memo 로 감싸주면 된다. 그렇다면 아래처럼 리랜더링 되지 않는 모습을 확인할 수 있다.

<img width="400" alt="첫번째 컴포넌트 메모이제이션" src="https://github.com/ju-ju2/precamp_class/assets/71650663/fdb51169-baa4-485c-977d-c00387c19476">

<br /><br />

<span style="color: #ff5100;;">\*\*문제발생</span> : count 변수를 받는 Third 컴포넌트가 리랜더링 되면서 그
<b>자식 컴포넌트인 Fourth 컴포넌트까지 함께 리랜더링</b> 되고있다!!😱  
문제를 해결해기 위해 Fourth 컴포넌트까지 메모이제이션해주자.

```jsx
const Fourth = React.memo(() => {
  console.log("네번째");
  return (
    <div>
      <div>네번째 컴포넌트</div>
    </div>
  );
});
```

결과는 아래처럼 깔끔쓰하게 값을 받는 Third 컴포넌트만 리랜더링된다. 최적화 끝. 휴🤗

<img width="400" alt="네번째 컴포넌트 메모이제이션" src="https://github.com/ju-ju2/precamp_class/assets/71650663/da5d4a5b-78a4-42ee-9c58-9934f4515b09">

<br /><br /><br />
이번 상태관리 부분을 공부하면서 평소에 공부하고 싶었던 것들을 많이 찾아볼 수 있었다. 신기하게도 Context API 와 useReducer 훅에 대해 포스팅하고난 후, 지금 듣고있는 교육 2개에서 바로 개념이 나왔다. 교육이 둘다 빠르게 진행되는 터라 미리 예습하지 않았으면 멘붕올뻔 했는데 참 신기하고 다행이게도 의도치않게 복습개념으로 수업을 들을 수 있었다. 그리고 메모이제이션에 대해 더 공부가 필요할 것 같은데 더 필요한 개념들은 다음에 포스팅 해봐야겠다. 오늘도 좀 더 성장한 느낌으로 포스팅 끝😎
