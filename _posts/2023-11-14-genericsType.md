---
layout: post
title: 타입스크립트 - 제네릭타입
subtitle: 이게 무슨 언어람
categories: typescript
tags: [typescript, generics]
image: /assets/images/20231114_generics.png
---

#### 들어가며😃

---

프로젝트를 진행하면 타입스크립트를 거의 필수적으로 쓰고 있는데 기본적인 타입만 썼었다.
그래서 유독 제네릭 타입만 보면 오잉? 싶은 느낌이,,,
다음 프로젝트 들어가기 전에 시간적인 여유가 잠깐 있어서 타입스크립트 강의를 듣고있는데 제네릭 타입이 예전처럼 어렵지도 않게 느껴졌고, 그리고 앞으로 더 가까워지려고 기록하는 글

<br/><br/>

#### 제네릭이란?

---

제네릭은 타입을 마치 함수의 파라미터처럼 사용하는 것을 의미한다. 타입을 사용할때 타입을 정의하는 것이다. 아래의 기본 틀을 보자.

- 일반 함수에 제네릭 사용

```typescript
function genericReturnFunc1<T>(arg: T): T {
  return arg;
}
```

- 화살표 함수에 제네릭 사용

```typescript
//  const genericReturnFunc2 = <T>(arg: T): T => {
//  return arg;
//  } // 에러남

// tsx에서 <>는 태그를 나타낼 때 사용하므로, 컴파일러에게 태그가 아닌 제네릭임을 알려 주어야 한다.
const genericReturnFunc2 = <T extends unknown>(arg: T): T => {
  return arg;
};
```

- 문법 요약

1.  `<T>` : 어떤 타입을 쓸 것인지 정의/ 제네릭을 쓸꺼야라고 선언!
2.  `(text: T)`: 받는 파라미더의 타입으로 쓸 꺼야
3.  `T`: 받았던 파라미터 값으로 반환할 꺼야

<br/><br/>

이제 함수를 불러올때 타입을 지정해주면 된다.
만약 인자와 타입이 맞지 않다면 에러가 날 것 이다.

```typescript
const str = genericReturnFunc2<number>(123); //o
const num = genericReturnFunc2<string>("123"); // o

const err = genericReturnFunc2<string>(123); // x
```

<br/><br/><br/>

#### 제네릭의 이점?

---

그러면 제네릭 타입은 왜 쓰는 것일까? 유니온 타입을 사용하여 비교해보자

```typescript
function generics(text: number | string) {
  console.log(text);
  return text;
}
```

```typescript
const abc = generics("abc");
// const abc = generics('abc').split('') // 에러남
```

**유니온 타입으로 타입을 지정했을 때의 문제점**

1. 위와 같은 코드에서는 a는 타입이 string이 아닌 `number | string로` 지정이 된다.
2. string 타입에서 쓸 수 있는 split 함수를 사용하려 하면 에러가 난다. number 타입에는 쓸 수 없기 때문이다.

<br/>

**제네릭의 경우?**

```typescript
function generics<T>(text: T): T {
  console.log(text);
  return text;
}

const str = generics<string>(‘abc’)
str.split(‘’)
```

제네릭 타입으로 쓴다면 받은 인자와 반환 값이 하나로 정해지기 때문에 위와같은 에러가 생기지 않는다. 함수를 정의할때 타입정의를 비워두고 <span style="color: #ff5100;">인자가 들어오는 시점에 타입을 정의를 하기때문에 타입정의에 대한 이점</span>을 가져간다.

<br/><br/><br/>

#### Interface 지정하는 법

---

객체의 타입을 지정할때 `interface/type` 을 쓴다. 여기에도 제네릭 타입을 지정하는 모양은 같다.

```typescript
Interface GenericType<T> {
value: T
selected: boolean
}
const obj: GenericType<string> = {value: ‘abc’, selected: false}
const obj: GenericType<number> = {value: 123, selected: false}
```

위처럼 받을 타입을 꺽쇠에 지정하면 끝! 아쥬 쉽쥬?😎

<br/><br/>

<span style="color: #ff5100;">+ 유니온 타입에서 제네릭으로 업글</span>

```typescript
function testFunc = (item: GenericType<string> | GenericType<number>) => {
   ///
}
->
function testFunc<T> = (item: GenericType<T>) => {
   ///
}
```

인터페이스 타입을 지정해주었어도 함수를 정의할때 다시 제네릭형(?)으로 응용한다면 유니온형을 피할 수 있다.

<br/><br/>

<span style="color: #ff5100;">+ 타입 확장</span>

```typescript
interface Dropdown<T> {
  value: T;
  title: string;
}

interface Detailed<K> extends Dropdown<K> {
  description: string;
  tag: K;
}
```

이렇게 되면 Detailed라는 타입에 value, title이 추가되는 것이고 value와 tag는 같은 제네릭 타입을 가지게 된다. 이렇게 이중으로 겹쳐서 사용가능하다.

```typescript
const detailedItem: Detailed<string> = {
  title: "abc",
  description: "aabbcc",
  value: "def",
  tag: "defied",
};
```

<br/><br/><br/>

#### 타입 제한

---

- 제네릭 타입 제한1 - `extents`

```typescript
const textLength = <T extends unknown>(text: T): T => {
  console.log(text.length); // 에러
  return text;
};
```

원래 위의 코드에서는 text의 타입이 정해져있지 않기 때문에 text.length에서 에러가 난다.
그래서 타입을 확장해주고 length라는 속성을 제한시키며 에러를 없앨 수 있다.

```typescript
interface LengthType {
  length: number;
}
```

```typescript
const textLength = <T extends LengthType>(text: T): T => {
  console.log("length : ", text.length);
  return text;
};

textLength<string>("hello"); // length : 5, string 속성의 length 사용 가능
textLength({ length: 3 }); // length : 3 , 파라미터 정보의 length 사용 가능
```

<br/><br/>

- 제네릭 타입 제한2 - `extents keyof`

타입의 키로만 파라미터 사용 가능하도록 제한둔다.

```typescript
interface ShoppingItem {
  name: string;
  price: number;
  stock: number;
}
```

```typescript
const getShoppingItem = <T extends keyof ShoppingItem>(
  itemOption: T
): ShoppingItem[T] => {
  const shoppingItem: ShoppingItem = {
    name: "example",
    price: 0,
    stock: 3,
  };

  console.log("itemOption", shoppingItem[itemOption]);
  return shoppingItem[itemOption];
};

// getShoppingItem(10); // 에러: 인자로 숫자가 올 수 없다.
getShoppingItem("name"); // example
getShoppingItem("price"); // 0
```

<br/><br/><br/>

---

타입스크립트는 거의 필수이기 때문에 한번 정리하고 싶었는데 내가 알고 쓰고 있는 것들은 너무 기초적인 것들이라 막상 정리하기엔 너무 애매했는데 이렇게 짬내어 정리할 수 있어 다행이다. 제네릭 타입을 볼때마다 눈이 돌아갔는데 이번 정리를 통해 집중해서 무슨 코드인지 더 잘 볼 수 있을 것같고, 그동안 유니온 타입 사용으로 겪어왔던 타입 에러에 대한 문제 이해를 더 잘 할 수 있을 것 같다는 기대감이 온다,,,후~🐥🏋🏻
