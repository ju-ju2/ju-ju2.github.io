---
layout: post
title: 타입스크립트 - 유틸리티타입
subtitle: 이게 무슨 언어람
categories: typescript
tags: [typescript, utility]
image: /assets/images/20231127_utility.png
---

#### 들어가며😃

---

이전에 정리한 제네릭 타입에 이어 유틸리티 타입을 정의해보려한다.

<br/><br/>

#### 유틸리티 타입이란?

---

유틸리티 타입이란 이미 정의된 타입을 변환할때 쓰는 타입 문법이다.

```typescript
interface MyType {
  name: string;
  age: number;
  gender: number;
  height: number;
}
```

<br/>

예를 들어 MyType 이라는 타입을 지정했는데 어떤 객체에는 age가 없는 3개의 타입만 필요하고, 어떤 객체에는 age 와 name 이 필요할 수 있다. 이런 경우에 지정된 타입을 입맛대로 쏙쏙 골라 타입을 지정해줄 때 사용하는 것이 유틸리티 타입이다. 그리고 오늘은 가장 많이 사용되는 <span style="color: #ff5100;">타입 3가지를 정리</span>해보려고 한다.

<br/><br/>

#### 😀Partial(파셜타입)

---

**특정 타입의 부분집합을 만족하는 타입**

```typescript
interface Address {
  email: string;
  address: string;
}

type MyAddressType = Partial<Address>; // 파셜이라는 타입스크립트에서 기본적으로 제공해주는 타입을 사용하기때문에 interface로 선언 불가

const empty: MyAddressType = {}; // 가능

const email: MyAddressType = {
  email: "test@abc.com",
}; // 가능

const all: MyAddressType = {
  email: "test@abc.com",
  address: "Seoul",
}; // 가능
```

위처럼 Address의 어떠한 구성이든 부분집합이라면 쓸 수 있다. 사실상 아래의 타입처럼 지정해준 꼴이다!

```typescript
type MyAddressType = {
  email?: string | undefined;
  address?: string | undefined;
};
```

<br/><br/>

#### 😀Pick

---

**몇개의 속성을 선택**

```typescript
interface MyType {
  name: string;
  skill: string;
}
const pickTest: Pick<MyType, "name"> = {
  name: "한쥬쥬",
  // skill: '프론트엔드 개발'
  // 에러 : 'Pick<MyType, "name">' 형식에 'skill'이(가) 없습니다.
};
```

<br/>

위에서는 MyType 중 name 만 선택하여 타입으로 지정하였기에 객체안에 skill을 선언하면 에러가 난다.

#### 😀Omit(오밋)

---

**지정된 타입에서 특정 속성만 제거**

```typescript
interface MyType {
  name: string;
  phone: number;
  address: string;
  company: string;
}
const myInfo1: Omit<MyType, "address"> = {
  name: "한쥬쥬",
  phone: 12345678,
  company: "한국",
};
const myInfo2: Omit<MyType, "address" | "company"> = {
  name: "한쥬쥬",
  phone: 12345678,
  // company: '한국' // 에러 : 개체 리터럴은 알려진 속성만 지정할 수 있으며 'Omit<MyType, "address" | "company">' 형식에 'company'이(가) 없습니다.
};
```

<br/><br/>

---

회사에서 프로젝트를 진행하면서 타입이 중복되는 것을 많이 느꼈다. 개발 중 도중에 타입이 달라질 수 있기때문에 우선 모든 api 와 변수 마다 타입을 하나씩 중복과 상관없이 지정해주었다. 그런데 개발을 하면서 이건 정말 응용해서 쓰면 되겠다 싶은 것들이 있었다. 이전에 포스팅하면서 extends에 대해 익혔고 오늘 유틸리티 타입을 정리해보았으니 이것들을 앞으로 응용해보아야겠다. 후후후
