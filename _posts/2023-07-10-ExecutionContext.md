---
layout: post
title: 자바스크립트 실행 컨텍스트와 렉시컬 환경
subtitle: 호이스팅 정의는 덤이다.
categories: javascript
tags: [javascript]
image: /assets/images/2023-07-10-context/context.png
---

#### 들어가며😃

---

프리온보딩 인턴십 중 강사님이 Class 와 함수를 비교하면서 동작+상태 가 필요할때는 Class, 동작만 필요하면 함수를 쓰는게 효율적이라고 했다. 함수에서 값을 기억하려면 Closure를 써야한다 했는데 나는 여기서 클로저에 꽂혔다⚡️. 그런데 말이다,,,강사님이 한 말이 어떤 의미인지 알고싶어 클로저에 대해 공부하려했던 것인데,,, 클로저 > 스코프 > 호이스팅 > 렉시컬 환경 > 실행컨텍스트 까지 거슬러 올라와버렸다..😵‍💫 이런 비하인드 스토리와 함께 오늘은 클로저를 정리하기전에, 꼭 알아두어야하는 개념인 실행컨텍스트와 렉시컬 환경에 대해 먼저 포스팅해보려한다.

<br/><br/>

### 실행 컨텍스트(Execution Context)란?

---

<span style="color: #ff5100;;">실행 컨텍스트란 자바스크립트 코드가 실행되는 환경</span>이다. 함수가 실행되면 해당 함수 실행에 대한 실행 컨텍스트가 생성되고, 자바스크립트 엔진에 있는 콜스텍에 순서대로 쌓인다. 스택(Stack)의 경우 FILO(First In, Last Out)구조이기 때문에 순서가 보장되고, 콜스택 내부에 쌓인 실행 컨텍스트의 정보를 통해 환경을 보장할 수 있는 것이다. 음,, 이해하기 어려울 수 있다. 아래의 코드와 이미지를 통해 실행 컨텍스트가 생성되고 사라지는 과정을 알아보자.  
<br />

```javascript
console.log("스크립트 실행!");

function B() {
  console.log("B 실행!");
}

function A() {
  console.log("A 실행,");
  B();
  console.log("B 종료!, 남은 로직 실행!");
}

A();
console.log("A 종료!, 남은 로직 실행!");

// (1) 스크립트 실행!
// (2) A 실행,
// (3) B 실행!
// (4) B 종료!, 남은 로직 실행!
// (5) A 종료!, 남은 로직 실행!
```

<img width="600" alt="실행 컨텍스트 생성 과정" src="https://github.com/ju-ju2/precamp_class/assets/71650663/bbf18e87-69a2-44ba-aea1-c4db1bf2f025">

<br/>

1. 우선 **전역 컨텍스트는 자바스크립트를 실행하는 순간, 콜스택에 담긴다.** 브라우저의 경우 window, localStorage, Math 와 같은 객체를 사용할 수 있는 이유다.
2. 함수 A가 실행됨에 따라 **a실행 컨텍스트를 생성하고 콜스텍에 push**한다. 전역 콘텍스트에 있던 로직은 일시적으로 중단된다.
3. 함수 B가 실행됨에 따라 **B실행 컨텍스트를 생성하고 콜스텍에 push**한다. A의 로직은 일시적으로 중단된다.
4. 함수 B가 종료되고 **B실행 컨텍스트가 콜스텍에서 POP 제거**된다. A 컨텍스트가 최상단에 있게 되고 남은 로직들이 수행된다.
5. 함수 A가 종료되고 **A실행 컨텍스트가 콜스텍에서 POP 제거**된다. 전역 컨텍스트가 최상단에 있게 되고 남은 로직들이 수행된다.
6. 전역에 있는 코드가 마지막 라인까지 모두 실행이 되면 **전역 컨텍스트도 사라진다.**

<br/>

여기서 중요한 포인트는 <span style="color: #ff5100;;">함수 실행 컨텍스트는 함수가 선언될 때가 아니라 실행될 때! 생성</span>되는 것이다. 위의 코드 구조를 보면 **B가 먼저 선언되었지만 콜스택에 쌓이는 건 먼저 실행된 A**이다!

<br/><br/>

### 호이스팅

---

아래에서 나올 환경 레코드와 관련하여 먼저 알아두어야 할 개념이다.

**호이스팅(Hoisting):** <span style="color: #ff5100;;">선언 라인 전에도 에러가 나지 않고, 변수를 참조할 수 있는 현상. </span>선언문이 마치 최상단에 끌여올려진 듯한 현상
호이스팅 현상이 발생하는 이유는 선언문이 있는 코드라인을 물리적으로 최상단에 끌어올린 것이 아니라, **자바스크립트 엔진이 먼저 전체 코드를 스캔하면서 변수같은 정보를 실행컨텍스트 어딘가에 미리 기록(record)**해두기 때문이다.

<br />

- **변수 호이스팅 (Variable Hoisting)**

  `생성단계` : 본격적인 실행에 앞서 **선언문을 스캔하고 환경 레코드에 미리 기록하는 과정.**  
   자바스크립트 엔진은 코드를 실행하면 우선 전역 컨텍스트를 생성해서 콜스택에 쌓는다. 그 후 전체를 스캔하면서 선언할 것이 있다면 찾아보고 먼저 선언해 둔다. 위의 코드를 참조한다면, 실행 컨텍스트 안에 있는 환경레코드에 새로운 식별자 hello 를 기록해둔다. 그리고 var 로 선언하였기 때문에 undefined로 값을 초기화 해두는 것이다.

  - 선언(Declaration) : 메모리 공간을 확보하고 식별자와 연결

  ```
  {hello      }
  ```

  - 초기화(Initialization) : 식별자에 암묵적으로 undefined 값 바인딩

  ```
  {hello : undefined}
  ```

  <br />

  `실행단계` : 선언문 외에 나머지 코드를 순차적으로 실행하는 과정. 필요한 경우 생성 단계에서 환경 레코드에 기록해둔 정보를 참고하거나 업데이트 하게 된다. 아래의 코드에서 실행 단계의 순서를 따져보면 아래와 같다.

  <img width="400" alt="렉시컬 환경 - 변수 var" src="https://github.com/ju-ju2/precamp_class/assets/71650663/7426280a-5907-4175-9a9b-039b6efbe5a5">

1. **console.log(hello)**: hello의 값을 출력하는 console.log가 실행된다. 값을 출력하려면 hello에 바인딩 된 값이 뭔지 알아야하는데 자바스크립트는 현재 활성화된 실행 컨텍스트 내에 환경레코드를 보고 이미 기록된 hello의 값(undefined)을 참조해서 문제없이 값을 출력해낸다.
2. **var hello = "안녕";** : 환경 레코드에 있는 hello 식별자를 undefined 에서 "안녕" 이라고 업데이트한다. {hello: "안녕"}
3. **console.log(hello)**: 환경 레코드를 참조해서 hello의 값은 "안녕"으로 출력한다.

<br/>
<em style='font-size: 20px; color: #BAD1C2; font-weight: bold;'>만약 let, const 키워드로 변수를 선언한다면? 🤷🏻‍♀️</em>

<img width="400" alt="렉시컬 환경 - 변수 const" src="https://github.com/ju-ju2/precamp_class/assets/71650663/8683e560-4749-4c29-a880-07fbc6b0df14">

이 경우 엔진이 hello라는 식별자를 기록!{hello} 까지만 하고 값을 undefined와 같이 초기화하지 않는다 . 따라서 선언문 이전에 hello를 참조하려하면 Reference Error가 발생하는 것이다.(일시적 사각지대)

<br/>

- **함수 호이스팅 (Function Hoisting)**

<img width="400" alt="렉시컬 환경 - 변수 const" src="https://github.com/ju-ju2/precamp_class/assets/71650663/8b79ff64-1f0a-4c89-9b33-e325773d5043">

자바스크립트는 함수를 변수에 담을 수 있다. var 키워드에 함수를 담아 선언문 이전에 실행하려고 하면 환경 레코드에 기록되어있는 study의 값은 undefined로 초기화 되지만 undefined라는 데이터 타입은 함수와 달리 호출될 수 없기 때문에 Type Error가 발생한다.

만약 함수를 let/const 키워드로 선언한다면 아직 환경레코드에 바인딩 된 값이 없어 Reference Error가 발생한다.
이렇게 **함수 표현식으로 함수를 선언하면 함수를 변수가 담고 있기 때문에 위에서 언급한 변수 호이스팅과 똑같이 동작하게 된다.**

<br/>
<em style='font-size: 20px; color: #BAD1C2; font-weight: bold;'>만약 함수표현식이 아닌 선언식으로 선언한다면? 🤷🏻‍♀️</em>

<img width="400" alt="렉시컬 환경 - 함수 선언" src="https://github.com/ju-ju2/precamp_class/assets/71650663/3d1fc0a5-8645-4796-af6c-387cc699c357">

자바스크립트 엔진이 함수 선언과 동시에 완성된 함수 객체를 생성해서 환경레코드에 기록해둔다. 그렇기 때문에 실행단계에 있어 선언 전에도 함수를 사용할 수 있다.

<br/><br/>

### 실행 컨텍스트의 구조 - 렉시컬 환경

---

위에서 호이스팅 개념을 잡아봤는데 이는 여기서 나올 환경 레코드와 연관이 있기 때문이다.  
렉시컬 환경은 `환경 레코드(Environment Record)`와 `외부 환경 참조(Outer Environment Reference)`로 이루어져 있다.

<img width="200" alt="렉시컬 환경" src="https://github.com/ju-ju2/precamp_class/assets/71650663/a8465e4e-6f0e-4a10-883e-ca8b8e78362d">

<br />

- **환경 레코드(Environment Record)**
  : <span style="color: #ff5100;;">스코프에 포함된 식별자를 등록하고, 등록된 식별자에 바인딩 된 값을 기록</span>한다. 위에서 자바스크립트 엔진이 먼저 전체 코드를 스캔하면서 변수같은 정보를 실행컨텍스트 어딘가에 미리 기록(record)한다고 했는데 이때 기록하는 곳이 환경 레코드인 것이다. 아래의 이미지처럼 기록하는 저장소.

<img width="400" alt="렉시컬 환경 - 변수 var" src="https://github.com/ju-ju2/precamp_class/assets/71650663/12cb7ebf-79c4-46a5-8345-ec820d63c969">

<br />

- **외부 환경 참조(Outer Environment Reference)**
  : 바깥 렉시컬 환경을 가리키는데 함수가 중첩되고 본인의 실행 컨텍스트에서 식별자를 찾을 수 없을 때 <span style="color: #ff5100;;">외부 실행 컨텍스트의 환경 레코드에서 식별자를 참조</span>하는 것이다.

<br/><br/>

<em style='font-size: 20px; color: #BAD1C2; font-weight: bold;'>식별자가 무엇일까? 🤷🏻‍♀️</em>

- **식별자 결정(Identifier Resolution)** : 코드에서 변수나 함수의 값을 결정하는 것

```javascript
let fruit = "바나나";
console.log(lamp); // "바나나"

function changeFruit() {
  let fruit = "포도";
  console.log(fruit); // "포도"
}
```

위와 같이 fruit의 값을 출력하려고 보니 "바나나" 와 "포도" 값이 두개가 있다. 이처럼 **콜스텍 안에 동일한 변수가 여럿일때, 그 값을 결정해내는 것을 식별자 결정**이라고 한다.

자 그럼 아래에서는 <span style="color: #ff5100;;">자바스크립트 엔진이 어떻게 outer를 통해 식별자를 결정하는지 알아보자.</span>

<br/>

<img width="600" alt="렉시컬 환경 - 함수 선언" src="https://github.com/ju-ju2/precamp_class/assets/71650663/05b091c3-fff4-4486-8dda-dd899cb9e915">
<br/>

1. firstFruit 실행 컨텍스트가 생성되고 fruit: 바나나를 환경 레코드에 저장한다.

2. SecondFruit 실행 컨텍스트가 생성되면서 **바깥 렉시컬 환경(firstFruit)으로 돌아갈 수 있는 outer를 만든다.** 이를 사용하여 필요한 경우 이전 실행 컨텍스트의 환경 레코드에 저장된 식별자도 참조할 수 있는 것이다.

3. ThirdFruit 실행 컨텍스트가 생성되었다. 해당 실행 컨텍스트에 fruit 값이 기록되어있지 않아 외부 **실행 컨텍스트를 차례대로 둘러본다.** SecondFruit 환경 레코드의 fruit 값 "포도" 를 가져온다. 그리고 이미 SecondFruit 에서 값을 찾았기 때문에 firstFruit 까지 가지 않는다.

4. console.log(animal) 을 실행하려하니 ThirdFruit 환경 레코드에 기록이 없다 -> SecondFruit 을 참조 해보지만 없다. -> firstFruit 을 참조해보지만 없다. -> Reference Error

<br/><br/>

### 실행 컨텍스트 재정리

---

결국 실행 컨텍스트는 식별자를 더욱 효율적으로 결정하기 위한 수단으로써 필요한 정보를 한데 모아 제공하는 객체라고 정리할 수 있겠다.

<br/><br/><br/><br/>

---

정리를 하려 깊이 들어가다보니 그냥 알고만 있던 개념의 깊은 원리를 깨달았다. 초반에 var, let, const 를 배울때 var 키워드는 호이스팅 이슈로 사용을 지양한다고 배웠다. 그런데 그 말에 이렇게나 넓은 범위과 관련 개념들이 존재하다니,, 오늘을 기점으로써 선언 키워드의 차이점은 무엇인지, 함수선언식과 함수표현식의 차이가 무엇인지 정확히 이해하였다. 오늘도 느끼는 무한하고 신비로운 자바스크립트 세계 😵‍💫😵‍💫

<br/><br/>
참고 : [[10분 테코톡] 💙 하루의 실행 컨텍스트](https://www.youtube.com/watch?v=EWfujNzSUmw)
