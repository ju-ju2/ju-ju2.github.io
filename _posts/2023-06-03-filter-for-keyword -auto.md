---
layout: post
title: filter 함수를 통한 키워드 자동 검색_1
subtitle: 검색버튼 없이 필터링
categories: project
tags: [react, filter]
image: /assets/images/2023-06-03-filter-auto/filter-auto.png
---

지난 포스팅에 이어 **검색(필터) 기능을 해주는 Search 컴포넌트 구현 과정**에 대해 더 풀어보려한다.
이전에서는 검색 버튼을 클릭하거나 엔터키를 눌러 받은 키워드를 필터링했다면 이번에는 키워드가 변할 때 마다 필터링을 해보겠다.
방법은 3가지가 있다.

1. 키워드가 변할 때 마다 필터링
2. 시간차를 두고 필터링
3. 2번을 쉽게 해주는 lodash 라이브러리 이용
   <br /><br />

이번 포스팅에서는 1번의 경우를 다뤄보려한다.
<br /><br /><br />

### - 키워드가 변할 때 마다 필터링

### 부모 컴포넌트

부모컴포넌트에서 이전과 마찬가지로 원래의 데이터(defaultData)와 필터링 된 데이터를 리패치하는 handleSearch 함수를 받아올 것이다.

```typescript
const [data, setData] = useState(defaultData);

const handleSearch = (filteredData: any) => {
  setData(filteredData);
};

return <>...</>;
```

<br /><br /><br />

### 필터 컴포넌트

그리고 필터 컴포넌트는 더 간단해졌다. (해당 부분에선 세션스토리지에 키워드를 입력하는 로직을 생략했다. 해당 방식에 대해 궁금하다면 [이전포스팅](https://ju-ju2.github.io/project/2023/06/01/filter-for-keyword.html){: target="\_blank" }을 참고해보길..!🌝)

```typescript
export default function Filter({
  defaultData,
  handleSearch,
  ...rest
}: FilterProps) {
  const [keyword, setKeyword] = useState("");

  const SearchKeyword = () => {
    const filteredData = defaultData.filter(
      (data) =>
        data.title.toLowerCase().includes(keyword.toLowerCase()) ||
        data.subTitle.toLowerCase().includes(keyword.toLowerCase())
    );
    console.log(`2: ${keyword}`);
    handleSearch(filteredData);
  };

  const onChangeKeyword = (e: ChangeEvent<HTMLInputElement>) => {
    setKeyword(e.currentTarget.value);
    console.log(`1: ${e.currentTarget.value}`);
    SearchKeyword();
    // 검색창이 비워졌을 때, defaultData로 다시 매핑
    if (!e.currentTarget.value) {
      handleSearch(defaultData);
    }
  };
  return (
    <SearchDiv>
      <StyledInput
        type="text"
        placeholder="검색어를 입력하세요."
        onChange={onChangeKeyword}
        {...rest}
      />
    </SearchDiv>
  );
}
```

코드를 살펴보면 onClick 이벤트나 엔터키 처리 과정이 없어지면서 로직은 더 간단해졌다.
그런데 여기서 문제가 있다!
<br /><br /><br />

![1한박자늦음](https://github.com/Side-Effect-Team/side-effect-frontend/assets/71650663/ed595b05-0c26-423e-93e4-076e427ae652)

어랏,,키워드를 잘 전달해주고 있는데 패치되는 데이터는 한박자 느리다. 로그를 살펴보면 onChangeKeyword에 전달되고 있는 키워드와 이 함수로부터 실행되는 SearchKeyword 함수로부터 찍히는 로그가 다른걸 확인할 수 있다.
<br /><br /><br />

이를 해결하기위해 `useEffect`를 사용하여 keyword 값이 변하면 SearchKeyword 함수를 실행하도록 변경해주었다.

```typescript
  useEffect(() => {
    SearchKeyword();
  }, [keyword]);

    const onChangeKeyword = (e: ChangeEvent<HTMLInputElement>) => {
      ...
    // SearchKeyword(); 삭제!
  };
```

<br /><br /><br />

결과는 짠!

![2해결](https://github.com/Side-Effect-Team/side-effect-frontend/assets/71650663/ecd4cee4-ee69-4c7f-8c83-8160e71bf6db)

키워드가 변함과 동시에 바로바로 필터링이 된다. 로그도 아주 동일하게 이쁘게 잘 찍힌다😎
<br /><br /><br />

그런데 말이다.. 이 방식은 **누가 봐도 비효율적일 것이다.** 입력값이 토시 하나 하나 변경될 때마다 SearchKeyword 함수가 실행된다면 데이터의 양이 늘어날 수록 작업량이 많아져 필터링이 느리게 될 것이다. 또한 유저들이 검색하고 싶은 키워드가 있다면 획 하나하나씩 필터링 해보려고 기다리진 않을 것이다.

이런 문제를 해결하기 위해선 입력값이 변경되고 일정 시간이 지난 후에 필터링 작업을 수행하도록 코드를 변경할 수 있을 것이다. 이를 `딜레이드 실행` 이라고 하는데 이 방법에 대해선 다음 포스팅에서 풀어보려한다. 그럼 오늘 포스팅은 여기서 끝!👋🏻
