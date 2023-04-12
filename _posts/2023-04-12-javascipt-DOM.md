---
layout: post
title: 리액트 DOM
subtitle: 돔돔 모든게 돔세상
categories: react
tags: [javascript, react]
image: /assets/images/2023-04-12/DOM.png
---

### DOM 이란

---

DOM은 문서 객체 모델(Document Object Model)로, HTML 문서의 모든 요소를 객체로 인식하여 프로그래밍적으로 제어할 수 있게 한다. 예를 들어, HTML 문서에서 <p> 요소, <div> 요소, \<input> 요소 등은 모두 DOM 요소이다. 이러한 DOM 요소들은 부모-자식 관계에 따라 계층 구조를 이루며, 이를 통해 웹 페이지를 구성한다.

### 가상DOM 이란

---

리액트에서는 가상 DOM(Virtual DOM)이라는 개념이 도입되어 있다. 사실 가상 DOM은 실제 DOM과 거의 동일한 개념이다. 그저 가상 DOM은 메모리상에 존재하는 가짜 DOM으로, 리액트에서 컴포넌트의 상태(state)나 속성(props)이 변경될 때마다 업데이트되고, 이를 통해 브라우저에 실제로 변경 사항이 적용되기 전에 렌더링을 최적화한다.

이게 무슨소리냐 하믄, 1.상태가 변경되면 리액트 컴포넌트 내의 가상 DOM 요소가 모두 다시 렌더링되고, 2.실제 DOM과 비교해서 변경된 부분만 뽑아서 업데이트한다는 것이다. 이렇게 되면 DOM 조작 횟수를 최소화하여 실제DOM에서 불필요한 렌더링을 줄이고, 성능을 향상시킬 수 있다.
<br />
<br />
<br />
<br />
<br />
<em style='font-size: 20px; color: #BAD1C2; font-weight: bold;'>아니, 돔 도작한다는게 뭔데?🤷🏻‍♀️</em>

DOM 조작이란, HTML 요소들을 자바스크립트를 사용하여 동적으로 변경하는 것을 말한다.
예를 들어 아래의 코드를 보자. 아래는 리액트를 사용하지않은 자바스크립트 코드다.

```html
<div id="container">
  <h1>Hello, World!</h1>
  <ul>
    <li>list 1</li>
    <li>list 2</li>
    <li>list 3</li>
  </ul>
  <button id="btn">Click me</button>
</div>

<script>
  const container = document.querySelector("#container");

  function updateUI() {
    container.innerHTML = `
      <h1>Hello, World!</h1>
      <ul>
        <li>list 1</li>
        <li>list 2</li>
        <li>list 3</li>
        <li>list 4</li>
      </ul>
      <button id="btn">Click me</button>
    `;
  }

  document.querySelector("#btn").addEventListener("click", () => {
    updateUI();
  });
</script>
```

위의 예시에서 updateUI 함수는 버튼 클릭 이벤트가 발생하면 호출되어 UI를 업데이트한다. 하지만, updateUI 함수에서는 container 요소의 innerHTML 프로퍼티를 변경하므로, **모든 요소가 다시 렌더링된다. 겨우 list 4 만이 추가됐는데 말이다!!** 이는 매우 비효율적인 방법이며, 매우 복잡한 UI의 경우, 성능 이슈를 유발할 수 있다.

하지만, 리액트에서는 가상 DOM을 사용하여 변경된 부분만 실제 DOM에 반영하므로, 이러한 문제를 해결할 수 있다.

<br />
<br />
<br />
<br />
<br />
<em style='font-size: 20px; color: #BAD1C2; font-weight: bold;'>아니, 어차피 리액트도 가상돔에서 리랜더링 되는거면 똑같은거 아니야??🤷🏻‍♀️</em>

가상 DOM 요소가 모두 다시 렌더링되는 것은 처음에는 비효율적으로 보일 수 있다. 그렇지만 엄연히 실제 DOM과 가상 DOM은 다르다.
실제 DOM은 브라우저에서 직접 조작되는 객체이고, DOM 조작은 브라우저의 연산을 필요로 한다. 이에 비해, 가상 DOM은 메모리에 존재하는 가짜 DOM이며, DOM 조작은 메모리에서 이루어지기 때문에 **훨씬 엄청 매우 빠르다**.

예를 들어 요소가 많고 복잡한 웹 페이지의 경우, 실제 DOM에서 조작이 많아질수록 성능이 저하될 가능성이 높다. 또한, DOM 요소가 리랜더링될 때마다 화면이 깜빡이는 현상도 발생할 수 있다. 따라서, 가상 DOM에서 빠르게 리랜더링하고 실제 DOM과 비교 후, 변경된 부분만을 다시 그리는 것이 훨씬 효율적인 것이다.
<br />
<br />
<br />
<br />
그렇다, React나 다른 Vue, Angular 등의 많은 라이브러리나 프레임워크에서 가상돔을 사용하는 이유는 명확했다.
