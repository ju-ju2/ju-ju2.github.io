---
layout: post
title: 금지어 판별하여 지워주기
subtitle:
categories: react
tags: [javascript, form]
image: /assets/images/2023-09-05-bannedWord/bannedWord.png
---

#### 들어가며😃

---

사용자가 입력하는 값은 어떤 값이 들어올지 모르기 때문에 항상 다양한 경우의 수를 생각해야한다. 사이트에서 글을 쓸때, 비속어를 쓰면 자동으로 지워지는 현상이 있고, 사용자는 교묘하게 다른 방식으로 해당 언어를 표현하는 경우를 많이 접할 수 있다. 오늘은 이 상황을 재현해보려한다. <span style="color: #ff5100;">input창에 불순한 언어(금지어)가 들어오면 지워지는 로직</span>을 만들어보자!

<br/><br/>

### 1. 금지어 리스트 생성

---

우선 아래와 같이 <span style="color: #ff5100;">금지어를 선언</span>한다.

```javascript
const BANNED_WORDS_LIST = "나쁜말1,나쁜말2,나쁜말3,나쁜말4";

// 모든 페이지에서 리스트를 공유할 수 있도록 localStorage에 저장
localStorage.setItem("bannedWordsList", BANNED_WORDS_LIST);
```

그리고 <span style="color: #ff5100;">localStorage에 해당 리스트를 저장</span>한다. 현재 예제에서는 단순히 선언을 해주었지만, 사실 API를 호출하여 금지어를 받아와야할 것이다. 모든 페이지에서 금지어 리스트를 불러오기 위해 API를 호출하는 것은 비효율적이기에 로그인 시 금지어리스트를 받아와 localStorage에 저장하는 구조를 선택했다.

<br/><br/>

### 2. 금지어 변환 함수 생성

---

```javascript
const getBannedWords = (value: string) => {
  const bannedWords = localStorage.getItem(`bannedWordsList`)?.split(",");
  const banned: string[] = [];
  let newValue = value;

  bannedWords?.forEach((word) => {
    if (newValue?.includes(word)) {
      newValue = newValue.replaceAll(word, "");
      banned.push(word);
    }
  });

  return { value: newValue, bannedWords: banned };
};
```

위의 함수는 input 창에서 값이 입력될 때 금지어가 포함되어있는지 판별한 후, 해당 금지어와 금지어를 지운 값을 리턴하는 함수이다. 구조를 자세히 들여다보면 아래와 같다.  
<br/>

- `const bannedWords = localStorage.getItem('bannedWordsList')?.split(",")`

  : localStorage에 저장해두었던 금지어 리스트를 가져와 배열로 만든다.

- `const banned: string[] = [];`

  : 리스트를 돌며 금지어가 있는지 판단하고 금지어가 있다면 해당 배열에 추가한다.

- `let newValue = value;`

  : 금지어를 지워주고 남은 부분을 리턴하기 위해 선언한다.

- ```
    bannedWords?.forEach((word) => {
      if (newValue?.includes(word)) {
        newValue = newValue.replaceAll(word, "");
        banned.push(word);
      }
    });
  ```

  : bannedWords 라는 금지어 리스트를 돌면서 각 단어가 입력된 값에 포함되어 있는지를 판별하고, **금지어가 포함되어 있다면 금지어 부분을 빈값("")으로 만들어준다. 그리고 해당되는 금지어는 banned 라는 배열에 push 한다.**

  <br/>

  <em style='font-size: 20px; color: #BAD1C2; font-weight: bold;'>반짝 회고 🤷🏻‍♀️</em>

  사실 처음에는 replace만 사용하여 함수를 구성했는데 사수님의 피드백에 따라 replaceAll로 변경하게 되었다. input창에 입력되는 값이라 그저 곧이 곧대로 입력하는 상황만 생각해서 금지어가 하나만 들어올 경우만 생각하고 replace만을 적용했었는데 복붙의 경우 ('나쁜말1나쁜말2')를 생각하지 못했었다. 그래서 banned라는 값도 스트링에서 리스트로 바뀌게 되었다.

- `return { value: newValue, bannedWords: banned }`

  :변경된 값 (입력: '나쁜말1이다' -> 반환: '이다') 과 해당 되었던 금지어(['나쁜말1'])을 반환한다.

<br/><br/>

### 3. onChange 이벤트 적용

---

```javascript
const onChangeNickname = (e: React.ChangeEvent<HTMLInputElement>) => {
  const { value, bannedWords } = getBannedWords(e.target.value);

  if (bannedWords.length > 0) {
    message.error(
      `해당 키워드는 입력하실 수 없습니다. (금지어 : ${bannedWords.join(", ")})`
    );
  }
  form.setFieldValue("nickname", value);
};

// ...생략
return(
    ...
     <Input
        placeholder="이름을 입력하세요"
        onChange={onChangeNickname}
    />
    ...
)
```

위에서 만들었던 getBannedWords 를 사용하기 위해 인자로 onChange 이벤트가 발생함에 따라 들어오는 값 (e.target.value)을 넣어준다. 그리고 그로 인해 리턴된 값을 아래와 같이 사용한다.

- **message 띄어주기**
  : 금지어가 지워지기 때문에 사용자에게 알람을 띄어주어야한다. 해당 알림을 위해 antd 라이브러리의 message를 사용했다. 그리고 위의 함수로 반환된 금지어 배열을 다시 쉼표(,)로 묶어 스트링으로 바꿔준 다음 알람을 띄운다.

- **input 값 변경하기**
  : 금지어가 지워진 새로운 값을 다시 input창에 배치시킨다. 해당 값을 useState를 통해 관리했다면 `setState()`를, form으로 관리했다면 `form.setFieldValue()`를 통해 값을 변환해준다.

<br/><br/>

### 4. 결과

---

<img width="600" alt="금지어 판별" src="https://github.com/ju-ju2/ju-ju2.github.io/assets/71650663/8b093558-5f67-4700-90da-07d8d529a685">

<br/>

위의 이미지처럼 금지어인 단어가 들어오자마자 지워지며 에러메세지를 띄어주는 모습 완성😎

<br/><br/>

---

휴,, 사실 금지어 로직을 구현하는데 우여곡절이 꽤 많았다. onChange 이벤트가 발생할 때 마다 판별하는 것은 너무 비효율적이라는 의견이 있어서 debounce 훅을 만들어도 보고, callback도 사용해보고 했지만 항상 사이드 이펙트가 있었다. 예를 들면, form을 사용하면서 validater를 통해 정의된 rule을 판별하여 에러메세지를 띄워줘야 하는데, 금지어가 들어가 값이 변경되어도 이전의 값을 인지하여 제대로 작동하지 않았다. 렌더링 때문에 시점 핀트가 안맞았다는 얘기,, 어쨋던 결국,, 돌고돌아 onChange 이벤트가 발생할때마다 금지어를 판별해주기로 해서 위의 로직을 완성하게 되었다. 이번 기회를 통해 useEffect를 사용한다고 해서 모든 상태를 완벽하게 맞출 수 없다는 점을 깨달았으니 좋은 경험이라고 말할 수 있겠다.🥲🚀
