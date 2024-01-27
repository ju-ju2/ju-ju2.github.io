---
layout: post
title: 타입스크립트 - 에러핸들링
subtitle: 꼭 알아야할 에러처리
categories: typescript
tags: [typescript, error]
image: /assets/images/20240127_errorHandling.png
---

#### 들어가며😃

---

정말 바쁜 하루하루를 보내고 있다. 새 프로젝트를 진행중에 있는데 비지니스적인 이슈가 너무 많아 일 이외에 다른 생각을 할 수 없는 일상이 되어버렸다,,,흑,,😵‍💫 이와중에 팀 내 스터디를 하고있는데 생산적인 느낌보단 조금 부담이 되어버렸다. 일할 시간도 없는데 무슨 스터디야!! 라는 생각이 들 참에 이번 챕터에서는 에러처리에 대해 공부하였고 굉장히 기본적이고 내가 제대로 배우지 않고 넘어갔던거라 꼭 기록하고 싶어서 남기는 글!

<br/><br/>

#### 에러 처리 방법

---

- _Error State_ : 예상할 수 있는 에러에 대한 상태 설정
- _Exception Error_ : 예상하지 못한 에러에 대한 처리

프로그래밍시 예상할 수 있는 에러(error state)인지 예상할 수 없는 에러(exption error)인지 <span style="color: #ff5100;">에러의 종류를 구분해서 처리할 필요가 있다.</span>

<br/><br/>

#### switch 문에서 에러처리 (feat.never)

---

```typescript
type Fruit = "apple" | "banana" | "melon";

const getFruit = (fruit: string) => {
  switch (fruit) {
    case "apple":
      console.log("사과 겟겟");
      break;
    case "banana":
      console.log("바나나 겟겟");
      break;
    case "melon":
      console.log("멜론 겟겟");
      break;
    default:
      throw new Error(`모르는 과일이야 ${fruit}`);
  }
};
```

위의 코드에서 getFruit 이라는 함수를 호출할 때 전달될 수 있는 모든 유니언 타입에 대해 각각의 케이스별로 모든 처리를 해야하고 이 케이스를 제대로 처리하지 않으면 에러가 발생한다.

<em style='font-size: 20px; color: #BAD1C2; font-weight: bold;'>그런데, 만약 개발자가 실수로 type은 추가하고 case 처리를 하지 않았다면?!🤷🏻‍♀️</em>
타입스크립트는 이런 핸들링 실수를 런타임이 아닌 컴파일 단계에서 발견할 수 있게 해준다.

```typescript
type Fruit = "apple" | "banana" | "melon" | "grapefruit";

const getFruit = (fruit: string) => {
  switch (fruit) {
    case "apple":
      console.log("사과 겟겟");
      break;
    case "banana":
      console.log("바나나 겟겟");
      break;
    case "melon":
      console.log("멜론 겟겟");
      break;
    default:
      throw new Error(`모르는 과일이야 ${fruit}`);
  }
};
```

위에서 'grapefruit'라는 타입을 추가하였고, 코드에서 핸들링을 제대로 해주지 않으면 앱 실행 시 에러가 뽝! 나타난다. 하지만 여기서 컴파일 시간에 수정할 수 있도록 만드는 것이 더 좋다! never 타입을 활용해보자!!

```typescript
///
default:
    const unknownFruit: never = fruit; // error
    throw new Error(`모르는 과일이야 ${fruit}`);
```

위에서는 케이스에 들어가지않고 default에 들어갈 수 있는것은 'grapeFruit'뿐이기에 invalid에 string 타입으로 잡히는 것이다. 좀 치는 타입스크립트 녀석,,,

```typescript
///
    case "grapeFruit":
      console.log("자몽 겟겟");
      break;
default:
    const unknownFruit: never = fruit; // error
    throw new Error(`모르는 과일이야 ${fruit}`);
```

'grapeFruit' 라는 케이스를 추가하게 되면 컴파일 에러가 발생하지 않는다. 똑똑한 타입스크립트!!! 발생할 수 있는 모든 케이스에 대해 처리를 했는것을 알고, default에 올 수 있는 타입은 never뿐인 것을 안다.

<br/><br/>

#### 에러 핸들링의 3단계\_try_catch_finally

---

```typescript
const getApple = (fruit: string): string => {
    if(fruit !== 'apple'){
        throw New Error(`${fruit} 은 사과가 아니야!`)
    }
    return '사과 인증서'
}

const fruitName = 'banana'
console.log(getApple(fruitName));
```

위의 코드에서는 fruitName 에 'apple'이 아닌 다른 것이 들어오는 순간 에러가 터지며 어플리케이션이 죽게된다.
<span style="color: #ff5100;">try~catch 구문을 적용한다면?!</span>
에러가 발생할 수 있는 함수(getApple())를 쓸때는 최대한 에러가 발생할 수 있는 정확한 부분에서만 try에서 감싸주면 된다.

```typescript
//
try {
  console.log(getApple(fruitName));
} catch (error) {
  console.log("사과 에러 발생");
}
```

위와 같이 에러처리를 하면 에러가 터져도 에러를 잡고 catch 문에서 로그를 찍어준다. 마지막까지 어플리케이션이 죽지않고 정상적으로 동작하는 것!

**<span style="color: #ff5100;">+++ finally</span>**

```typescript
//
try {
  console.log(getApple(fruitName));
} catch (error) {
  console.log("사과 에러 발생");
} finally {
  console.log("네 취향은 아오리사과🍏");
}
```

try 가 성공하던 실패하던 finally는 무조건 실행되게 되어있다. 무언가를 시도하고 결과에 상관없이 무언가를 실행해야한다면 finally에 넣어줘야한다. catch에서 무엇가를 처리할때 다른 에러가 발생하거나 혹은 리턴된다면 아래 구문이 실행되지 않기 때문에 try와 연관되거나 마무리할 것들이 있다면 꼭 finally 안에 넣어주는 것이 좋다.

<br/><br/>

#### Error State

---

에러가 나는 이유는 사실 한두가지가 아닐것이다. getFruit 함수에서 나타날 수 있는 에러라 하믄 인자가 사과가 아닐 수도 있고, 썩은 사과일 수 있고, 훔친 사과(?)일 수 있다. 그렇기 때문에 catch문에서 에러의 타입을 판단해 다른 로직을 사용할 수 있지 않을까..? 라고 생각할 수 있다만,,,아래와 같이..도전?

```typescript
//
try {
  console.log(getApple(fruitName));
} catch (error) {
  if (error instanceof StolenApple) {
    console.log("훔친 사과");
  }
  // error
}
```

하지만,,, try catch에 전달되는 error의 타입은 any 타입이다. catch로 에러를 받는 순간 any타입이 되어버리기 때문에 타입에 대한 정보가 사라져 위와 같이 타입 구분이 불가능하다.

그렇기 때문에 에러는 가급적 정말정말 예상하지 못하는 곳에서 사용하는 것이 좋고, **세부적인 에러를 처리하고 싶다면 Error State**를 사용하자!

```typescript
type ErrorState = {
  result: "fail";
  reason: "stole" | "expensive" | "rotten";
};

type SuccessState = {
  result: "success";
};

type ResultState = SuccessState | ErrorState;

const getFruit = (): ResultState => {};
```

getFruit 에서 알수없는 throw를 남발하는 것이 아닌 어떤 상태를 리턴하는지 상태를 타입으로 정의하는 것이 깔끔하게 프로그래밍할 수 있다.

<br/><br/><br/><br/>

---

이번 프로젝트 들어가면서 에러처리에 대한 로직을 짠적이 있는데 제대로 학습한 적이 없고 그냥 인강에서 하란대로 따라하고 넘어갔더니 혼자 에러처리를 하는데 멘붕이 왔었다. 그런데 이렇게 쉬웠다니,,,확실이 학습하고 정리를 하느냐 안하느냐에 따라 실력에 큰 차이가 나타나는구나. 사소한데에서 크게 느꼈다. 이번 기회로 정리할 수 있어서 너무 좋았고 앞으로 잘 써먹어야 겠다. 오늘도 지식 업업 🐥🐓🆙
