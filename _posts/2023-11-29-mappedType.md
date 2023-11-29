---
layout: post
title: 타입스크립트 - 맵드타입
subtitle: 이게 무슨 언어람
categories: typescript
tags: [typescript, mapped]
image: /assets/images/20231129_mapped.png
---

#### 들어가며😃

---

자바스크립트의 맵 메소드가 있다면 타입스크립트에는 맵드 타입이 있다니!

<br/><br/>

#### 맵드 타입(Mapped Type)이란?

---

이름에서 알 수 있다시피 자바스크립트의 map 메소드를 타입에 적용한 것이다.

- 기본문법
  `{ [ P in K ] : T }`

```typescript
type Fruits = "apple" | "banana" | "grape";
```

예를 들어, 과일 이름을 타입으로 지정하는 Fruits 타입이 있다고 하자.

그리고 각 과일의 이름에 숫자를 붙인 객체를 만들기 위해 Fruits의 타입들을 맵드 타입 문법을 이용해 키를 만들어주고 넘버 타입을 할당해준다.

```typescript
type FruitsAccount = { [K in Fruits]: number };

// type FruitsNum = {
// apple: number;
// banana: number;
// grape: number;
// } 와 같음

const fruitsInfo: FruitsAccount = {
  apple: 100,
  banana: 200,
  grape: 300,
};
```

위 코드에서 [K in Fruits] 구문은 for 구문과 유사하다!
앞에서 정의한 Fruits 타입의 문자열을 순회하면서 number 타입을 할당해주는 것이다.

<br/><br/>

#### 응용

---

기존의 정의 되어있는 타입을 부분집합으로 변환시키기.

```typescript
type Subset<T> = {
  [K in keyof T]?: T[K];
};

interface MyInfo {
  age: number;
  name: string;
}

const ageOnly: Subset<MyInfo> = { age: 20 };
const nameOnly: Subset<MyInfo> = { name: "Juju" };
const info: Subset<MyInfo> = { age: 20, name: "Juju" };
const empty: Subset<MyInfo> = {};
// const cityOnly: Subset<MyInfo> = { city: 'Jeju' }; // 에러

// 사실상 아래 타입과 같음
// interface MyInfo {
// age?: number;
// name?: string;
// }
```

`[K in keyof T]` 를 보면 기존의 정의 되어 있는 MyInfo라는 타입의 키를 가져오고, 그 키에 할당되어있는 타입까지 다시 재정의를 한다. 그렇지만 ?를 붙이기 때문에 있어도, 없어도 되는 부분집합이 가능한 것이다.

<br/><br/>

---

맵드 타입은 실 예제로 한번도 본적이 없어서 낯설긴 하다. 언젠가 실무에서 마주치게 된다면 꺼내 볼 글이 되기를..
처음 T, K, P 들로만 표현된 문법을 봤을때는 이게 뭐지 싶었는데 간단한 예제들로 살펴보니 훨씬 이해하기 쉬워졌다. 타입스크립트,,,내가 다 뿌순다
