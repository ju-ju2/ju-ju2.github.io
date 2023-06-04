---
layout: post
title: filter 함수를 통한 키워드 자동 검색_setTimeout_clearTimeout
subtitle: 딜레이 방식의 필터링
categories: project
tags: [react, setTimeout, clearTimeout]
image: /assets/images/2023-06-03-filter-auto/filter-auto.png
---

지난 포스팅에서는 키워드가 변할 때 마다 데이터를 필터링 했다. 하지만 작업량이 많아질 수록 필터링이 느려지는 이슈가 발생한다. 오늘은 해당 이슈를 해결하기 위한 딜레이드 필터링을 구현하려한다. 방식엔 두가지가 있다.

1. `setTimeout`,`clearTimeout` 내장함수를 이용한 딜레이 방식
2. 이를 쉽게 구현할 수 있는 lodash 라이브러리

오늘 포스팅에서는 팀 프로젝트에서 사용했던 1번 방식을 풀어볼 것 이다.

<br />
<div style='color: #d9d9d9;'>이전 포스팅에서는 전체데이터를 필터 컴포넌트로 불러와 필터링 된 데이터를 내보내는 방식을 구현했었는데 이번에는 키워드만 전달하는 방식으로 변경하려 한다!
input 창에 입력된 키워드 값을 전달해주고 부모컴포넌트에서는 전달받은 키워드를 변수로 받아 api 호출을 하기에 방식이 바뀌었지만 전체적인 필터 개념은 똑같다고 볼 수 있다.</div>

<br /><br /><br />
우선 차이를 보기 위해 인풋창에 들어온 값대로 키워드를 전달하는 방식을 보자.

### 실시간 검색

---

<br />

- 부모 컴포넌트

```typescript
export default function Test() {
  const [Keyword, setKeyword] = useState("");
  console.log(Keyword);

  return (
    <Container>
      <FilterWrapper>
        <Search setKeyword={setKeyword} />
      </FilterWrapper>
    </Container>
  );
}
```

- 서치 컴포넌트

```typescript
export default function Search({ setKeyword }: SearchProps) {
  const [searchKeyword, setSearchKeyword] = useState("");

  const onChangeKeyword = (e: ChangeEvent<HTMLInputElement>) => {
    setSearchKeyword(e.currentTarget.value);
  };

  useEffect(() => {
    setKeyword(searchKeyword);
  }, [searchKeyword]);

  return (
    <SearchDiv>
      <StyledInput
        type="text"
        placeholder="검색어를 입력하세요."
        onChange={onChangeKeyword}
      />
    </SearchDiv>
  );
}
```

<br /><br />
위의 코드는 `setKeyword`를 통해 키워드 값이 변할 때 마다 키워드를 할당한다.(키워드라 하기엔 매우 비효율적인 편🤦🏻‍♀️)

![1 setState만](https://github.com/Side-Effect-Team/side-effect-frontend/assets/71650663/5abb4374-e825-4a07-b9d4-fcfb13b02c3e)
<br />
(로그가 주륵주륵☔️)

이 방식에서 사용자가 키워드를 입력한 후 마지막으로 입력된 값만 보내주는 식으로 변경해보자.
<br /><br /><br />

```typescript
// 기존 useEffect 코드 변경
useEffect(() => {
  const timer = setTimeout(() => {
    setKeyword(searchKeyword);
  }, 300);
  return () => clearTimeout(timer);
}, [searchKeyword]);
```

코드를 살펴보면 `setTimeout` 과 `clearTimeout` 을 통해 키워드 할당 로직을 관리하고 있다. 해당 함수의 역할을 하나씩 분리해서 보자.
<br /><br /><br />

### setTimeout

---

첫 번째 인자로 전달된 콜백 함수를 두 번째 인자로 전달된 시간(밀리초)만큼 지연시킨 후 실행하는 함수.

```typescript
useEffect(() => {
  const timer = setTimeout(() => {
    setKeyword(searchKeyword);
  }, 300);
  // return () => clearTimeout(timer);
}, [searchKeyword]);
```

<br />

적용 결과 ↓

위의 코드 처럼 `clearTimeout`없이 `setTimeout`함수만 적용하면 키워드를 입력하면 지정된 시간 이후에 로그가 차례로 찍히게 된다.

![2 setTimeout만](https://github.com/Side-Effect-Team/side-effect-frontend/assets/71650663/88009880-4fab-4a4d-ae25-5764e9ddce2f)

### clearTimeout

---

clearTimeout은 setTimeout 함수에 의해 설정된 타이머를 취소하는 역할. clearTimeout 함수는 setTimeout 함수가 반환한 타이머 ID를 인자로 받아 해당 타이머를 취소한다.
<br /><br />

```typescript
useEffect(() => {
  const timer = setTimeout(() => {
    setKeyword(searchKeyword);
  }, 300);
  return () => clearTimeout(timer);
}, [searchKeyword]);
```

<br />

자 clearTimeout을 적용한 위의 코드를 정리해보자면,
`setTimeout` 함수는 setKeyword 함수를 300밀리초 후에 호출하는 일을 담당한다. 이를 통해 입력값이 변경된 후 일정 시간이 지난 후에 setKeyword 함수가 실행되도록 하는것이다! 그리고 `clearTimeout` 함수는 컴포넌트가 언마운트되거나 다음 useEffect 실행 전에 호출되어 이전에 예약된 타이머를 취소한다. 이는 입력값이 빠르게 변경되는 경우 이전에 예약된 타이머가 취소되고 새로운 타이머가 설정되어 최신 입력값에 대한 딜레이를 유지하는 것이다.
<br />
<br />
즉, 키워드를 계속해서 타이핑하고 있을 때 clearTimeout 이 타이머를 계속 취소하는 것이다. 그리고 타이핑이 끝나면 마지막 키워드 값을 setKeyword 에 저장하는 것이다.  
아래의 결과물을 보면 입력한 마지막 값만이 키워드로 전달되는 것을 볼 수 있다!😎

![3 모두적용](https://github.com/Side-Effect-Team/side-effect-frontend/assets/71650663/7e95ba16-eb24-4339-b0d5-abc586e3a3d1)

<br /><br /><br />

오늘은 키워드를 타이머 함수를 통해 관리하는 법을 정리해보았다.

이번 서치 컴포넌트 구현을 해보면서 해당 컴포넌트를 쓰는 이유에 대해 깊게 생각할 수 있었다. 단순히 키워드만을 제공하는게 아니라 효율적인 통신을 하도록 도와주는 역할, 사용자가 편리하게 검색할 수 있는 역할 등 부가적인 역할이 많았다. 그렇다,,컴포넌트가 존재하는데엔 다 이유가 있는 것 이였다,,

물론 아주 단순한 타이머 함수를 통해 상태 관리를 제어했지만 lodash 라이브러리를 사용하지 않았다는 점에서도 꽤 뿌듯했고 원리가 그렇게 어렵지 않구나를 알게되었다.  
+타이머 함수와 더 친해진 기분^^🌝 오늘도 뿌듯하게 포스팅 끝!
