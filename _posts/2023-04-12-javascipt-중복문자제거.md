---
layout: post
title: 자바스크립트 배열 중복 제거
subtitle: indexOf(), reduce() 함수 이해하기
categories: javascript
tags: [javascript, test]
---

javascript에서 배열의 중복되는 값을 제거하는 방식은 다양하다.  
그 중 `filter(),indexOf()` 함수를 이용하는 방법과 `reduce(),includes()` 함수를 이용하는 방법을 다루려고 한다. 우선 각 함수들의 개념을 정리하고 혼합하여 쓰는 방식을 정리하려 한다.

<h4 style="color: #ff5100;;">indexOf()</h4>

---

indexOf 함수는 특정 문자열을 검색하고, 검색된 문자열이 **처음으로 나타나는 위치 index를 리턴**하는 기능이다. 만약 문자열을 찾지 못했다면 -1를 리턴한다.

예시1. 문자열

```javascript
const str = "hahahoho";
let result = str.indexOf("ha");
// 결과 : 0 (0번째 인덱스에서 처음 발견)
let result = str.indexOf("ho");
// 결과 : 4 (4번째 인덱스에서 처음 발견)
let result = str.indexOf("wow");
// 결과 : -1 (발견하지 못함)
```

위의 예시를 보듯 "ha"와 "ho"는 str 문자열 내에 2번 존재하지만 처음 발견되었을 떄의 인덱스값만 반환한다.  
아래의 배열도 마찬가지다.
<br />
<br />
<br />
예시2. 배열

```javascript
const fruits = ["apple", "banana", "mango", "apple", "mango"];
let result = fruits.indexOf("mango");
// 결과 : 2

// ++ 만약 문자열을 뒤에서 부터 검색하고 싶으면 lastIndexOf() 함수를 이용하면 된다.
let result = fruits.lastIndexOf("mango");
// 결과 : 4
```

<br />
<br />
<br />

<h4 style="color: #ff5100;;">filter()</h4>

---

`filter()` 함수는 배열의 모든 요소를 검사하고 주어진 **조건에 부합하는 요소만을 새로운 배열로 반환**하는 함수다.
이 함수는 매개변수로 콜백 함수를 받는다.

예를 들어 [1,2,3,4,5] 라는 배열이 있다고 가정하고 짝수만 필터링해서 배열에 담고 싶다면 아래와 같이 진행하면 된다.

```javascript
const numbers = [1, 2, 3, 4, 5];

const evenNumbers = numbers.filter((number) => number % 2 === 0);
console.log(evenNumbers); // [2, 4]
```

구조를 살펴보자면 filter() 함수의 콜백 함수는 세 개의 매개변수를 가질 수 있는데, `filter((value, index, self)=>{})` 의 구성이다.
첫 번째 매개변수는 현재 요소의 값(value), 두 번째 매개변수는 현재 요소의 인덱스(index), 세 번째 매개변수는 배열 자체(self)이다. 하지만 self는 쓸 일 없다.
<br />
<br />
<br />

<h3 style="color: #ff5100;;">filter() 와 indexOf() 를 이용하여 배열의 중복 제거하기</h3>

---

자, 그렇다면 콜백함수의 조건을 필터링해주는 filter() 함수와, 조건에 부합하는 값의 처음 인덱스만 반환하는 indexOf()함수를 이용해서 배열의 중복을 없애보자.

```javascript
const arr = ["apple", "banana", "mango", "apple", "mango"];

const myArr = arr.filter((value, index) => {
  return arr.indexOf(value) === index;
});

console.log(myArr); // ['apple', 'banana', 'mango']
```

콜백함수 `(value, index) => { return arr.indexOf(value) === index; }` 의 원리는 간단하다.

1. arr 배열에 있는 순서대로 value 값이 들어온다.
2. arr.indexOf(value) 값에는 차례로 0, 1, 2, 0, 2 가 들어온다.
3. 각 value의 인덱스는 0, 1, 2, 3, 4 이다.
4. 2번 3번을 비교했을때 === 의 조건을 만족하는 것은 0, 1, 2 번째 인덱스 값들이다.
5. filter() 함수와 함께 새로운 배열이 생성된다.

이상이다.
<br />
<br />
<br />
<br />

<h4 style="color: #ff5100;;">reduce()</h4>

---

`reduce()` 함수는 배열에 사용되어 배열의 각 요소에 마다 주어진 콜백 함수를 실행하고, **하나의 결과값을 반환**하는 함수이다. reduce() 함수의 콜백함수는 다음과 같이 구성된다.  
`(누적된 값, 현재 value, 현재 index, 현재 배열)=>{}`

쉽게 예를 들자면 reduce() 함수로 배열 요소들의 합을 구할 수 있다.

```javascript
const numbers = [1, 2, 3, 4, 5];

const sum = numbers.reduce((accumulator, currentValue) => {
  return accumulator + currentValue;
}, 0);

console.log(sum); // 15
```

위를 간단히 설명하자면,

1. 초기값 0/ 0이 accumulator(누적된 값)에 들어감
2. 배열을 돌면서 누적된 값에 첫번째 요소 1을 더한다. => 누적 값 1
3. 누적된 값에 두번째 요소 2를 더한다 => 누적 값 3
4. 반복
<br />
<br />
<br />
<h4 style="color: #ff5100;;">includes()</h4>

---

includes() 함수는 배열에서 **특정 요소가 있는지** 여부를 확인하는 함수이며 `arr.includes("찾을 값", "검색시작 인덱스")` 로 구성된다. 결과는 boolean값을 리턴한다.

```javascript
const fruits = ["apple", "banana", "orange"];

console.log(fruits.includes("apple")); // true
console.log(fruits.includes("kiwi")); // false
```

<br />
<br />
<br />
<h3 style="color: #ff5100;;">reduce() 와 includes() 를 이용하여 배열의 중복 제거하기</h3>

---

자, reduce() 함수와 includes() 함수를 이용해서 배열의 중복을 제거해보자. 바로 예시 들어감.

```javascript
const fruits = ["apple", "banana", "orange", "apple", "kiwi", "banana"];

const myFruits = fruits.reduce((sum, value) => {
  if (!sum.includes(value)) {
    sum.push(value);
  }
}, []);

console.log(myFruits); // ['apple', 'banana', 'orange', 'kiwi']
```

위의 로직도 간단하다.

1. 초기값 빈배열 [] 설정됨 => sum에 [] 할당
2. reduce()함수가 배열을 순환하면서 value값을 가져온다.
3. !sum.includes(Value) : sum 은 현재 빈 배열이기 때문에 첫번째 value 값인 "apple"이 없다. sum에 값 푸시
4. sum은 ["apple"]이 되고 다음 "banana" 검증
5. 반복

아참 위의 로직에서 if문은 삼항 연산자로도 표현할 수 있다.
`sum.includes(value) ? sum : [...sum, value]`

굿쟙👍🏻
