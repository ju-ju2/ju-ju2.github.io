---
layout: post
title: 자바스크립트의 비동기 처리와 이벤트 루프
subtitle: 브라우저의 동작 원리를 알아보자-2
categories: CS
tags: [HTTP, CS]
image: /assets/images/20240826_eventloop.png
---

자바스크립트는 **싱글스레드 언어**로, 한 번에 하나의 작업만 처리할 수 있다. 그러나 비동기 작업을 지원하기 위해 이벤트 루프와 Web API 같은 여러 메커니즘이 사용된다.

<br/>
<br/>

### 1. 콜백 함수 (Callback Function)

---

콜백 함수는 다른 함수의 인자로 전달되어, 특정 시점에 호출되는 함수이다. 콜백 함수는 동기적(synchronous)으로 호출될 수도 있고, 비동기적(asynchronous)으로 호출될 수도 있다.

- 동기적 콜백 함수 예시

  ```
  function greeting(name) {
      console.log(`Hello, ${name}!`);
  }

  function processUserInput(callback) {
  const name = 'hanjuss'
  callback(name);
  }

  processUserInput(greeting);
  ```

  <br/>

- 비동기적 콜백 함수 예시

  ```
  console.log('1번 출력');
  setTimeout(() => {
    console.log('2번 출력');
  }, 1000);
  console.log('3번 출력');
  ```

  실행 순서 : 1 → 3 → 2
  왜 1 → 2 → 3 이 아닐까? 위의 실행 순서를 이해하려면 자바스크립트 엔진 구조를 이해할 필요가 있다.

  <br/>
  <br/>

### 2. 자바스크립트 엔진 구조

---

<br/>
<em style='font-size: 17px; color: #ff5100; font-weight: bold;'>힙(Heap)</em> : 메모리 할당이 이루어지는 공간으로, 객체와 변수가 저장된다.

<br/>
<em style='font-size: 17px; color: #ff5100; font-weight: bold;'>스택(Stack)</em> : 함수가 호출되는 순서대로 쌓이며, 후입선출(LIFO) 방식으로 작동한다.

- 스택 구조 예시

  ````
  function first() {
  console.log("첫 번째 함수 실행");
  second();
  console.log("첫 번째 함수 종료");

  }

  function second() {
  console.log("두 번째 함수 실행");
  }

  first();```
  ````

    <br/>
    <img src="https://github.com/user-attachments/assets/64c1a682-b741-4769-917d-d8ada9ff310f" width="500"/>

    <br/>

  `first()` 함수가 호출되면 스택에 쌓이고, 그 내부에서 `second()` 함수가 호출된다. 실행이 완료되면 스택에서 함수가 제거된다.

<br/>
<br/>

### 3. 싱글스레드와 비동기 처리

---

자바스크립트는 싱글스레드 언어로, 한 번에 하나의 작업만 처리할 수 있다.
!하지만, 웹 브라우저는 **Web API**, **이벤트 루프(Event Loop)**, 콜백 큐(Callback Queue)를 통해 비동기 작업을 처리할 수 있다.

<br/>
<img src="https://github.com/user-attachments/assets/e204c3a2-67ea-484e-b06a-3b7b34a4fa04" width="400"/>

<br/>

Web API 들은 모두 비동기 메서드이기 때문에 작동을 마치고 콜백함수를 콜백 큐에 넣는다. 거기서 콜백 함수들이 실행을 대기한다.

정리하자면 <u>자바스크립트 엔진은 싱글스레드지만 자바스크립트가 실제로 구동되는 환경인 웹 브라우저에는 여러개의 스레드가 사용된다. 명확히는 Web API 가 멀티 스레드로 동작</u>하는 것이다.
자바스크립트가 이것들과 상호 연동을 하기 위해 필요한 장치가 콜백 큐, 이벤트 루프인 것 이다.

<br/>
<br/>

### 4. 이벤트 루프 (Event Loop)

---

이벤트 루프는 자바스크립트 엔진 뒷편에서 일어나는 문맥의 일부로써 동작하는 하나의 장치이다.
**이벤트 루프는 호출 스택과 콜백 큐를 계속해서 주시**하고 있다. 호출 스택이 비어있으면, 먼저 들어온 순서대로 콜백 큐에 있는 콜백 함수들을 호출 스택으로 집어넣는다.

- 처리 예시

  ```
  function second() {
  setTimeout(() => {
      console.log('나는 바나나')
  },2000)
  }

  function first() {
      console.log("나는 토마토");
      second();
      console.log("나는 달걀");
  }

  first();
  ```

  <br/>
  <img src="https://github.com/user-attachments/assets/aba11f7a-87b2-4938-be54-7303e17a0329" width="400"/>

  <br/>

  1️⃣ first() 함수가 호출된다. first() 함수가 콜 스택에 쌓인다.

  2️⃣ first() 함수 안에서 console.log("나는 토마토")가 실행되고 콜스택에서 제거된다.

  3️⃣ first() 함수 안에서 second() 함수가 호출되고 콜 스택에 쌓인다.

  4️⃣ second() 함수 안에서 setTimeout 함수가 호출된다.

  5️⃣ setTimeout 함수는 비동기 함수이기 때문에, 콜 스택이 아닌 웹 API로 이동한다. 이때 타이머와 콜백함수가 분리된다.

  6️⃣ second() 가 리턴된다.

  7️⃣ first() 함수의 나머지 코드가 실행된다.

    <br/>
    <img src="https://github.com/user-attachments/assets/efd1a2e3-5501-4724-8f55-d385590ae7aa" width="400"/>

    <br/>

  8️⃣ 타이머가 종료되면, 웹 API에 있던 setTimeout의 콜백 함수가 콜백 큐로 이동한다.

    <br/>
    <img src="https://github.com/user-attachments/assets/85694a37-dcab-467f-9f6d-26ae7742ec15" width="400"/>
    
    <br/>

  9️⃣ 콜스택이 비어있으면 이벤트 루프가 콜백 큐에 있는 콜백 함수를 콜 스택으로 이동시킨다.
