---
layout: post
title: filter 함수를 통한 키워드 검색
subtitle: 검색버튼, 엔터키로 필터링
categories: project
tags: [react, filter]
image: /assets/images/2023-06-01-filter/filter.png
---

프로젝트에서 키워드를 검색하면 필터링해주는 필터 컴포넌트 구현을 맡았다.
데이터를 불러오는 방식이 전체 -> 무한스크롤 로 변경이 되기도 하고 키워드를 받는 api가 개발되어 필터 방식도 filter 함수로 쓰는 법 -> 키워드만 전달하는 법 으로 바껴 해당 방식은 결국 쓰이진 않게 되었지만..🥲
그래도 필터함수와 더 친해진 느낌이랄까..? 어쨋던 필터 컴포넌트를 구현해보고 변경된 과정 그리고 겪었던 에러&해결과정을 기록하려한다.

<img src="https://github.com/Side-Effect-Team/side-effect-frontend/assets/71650663/e9bb25f4-276d-4fd7-b3ca-5fe70e57b8e8" width="300"/>

위는 팀 프로젝트에서 사용되는 카드 컴포넌트이다. 필터링한다면 제목과 내용이 필터링 될 수 있겠다!
우선 Filter 컴포넌트를 사용하는 부모 컴포넌트에서 defaultData, handleSearch 함수를 받아와야한다.

```typescript
export default function Parent () {

    const [data, setData] = useState(defaultData); //api 통신을 통해 받아온 데이터
    const handleSearch = (filteredData: BoardCardProps[]) => {
    setData(filteredData); // Filter 컴포넌트에서 넘겨준 데이터로 다시 화면 구성

  };

    return (
        ....
        <Filter defaultData={defaultData} handleSearch={handleSearch}/>
    )
}
```

그리고 아래 Filter 컴포넌트

```typescript
export default function Filter({
  defaultData,
  handleSearch,
  ...rest
}: FilterProps) {
  // 인풋창에 입력될 키워드
  const [keyword, setKeyword] = useState("");

  // onChange 값에 들어오는 값들을 keyword에 할당
  const onChangeKeyword = (e: ChangeEvent<HTMLInputElement>) => {
    setKeyword(e.currentTarget.value);
  };

  // 부모 컴포넌트로부터 받아온 data에 위에서 입력한 키워드 값이 들어 있는 것들만 필터링
  const onClickSearch = () => {
    const filteredData = defaultData.filter(
      (data) =>
        data.title.toLowerCase().includes(keyword.toLowerCase()) ||
        data.content.toLowerCase().includes(keyword.toLowerCase())
        )
    );
    // 데이터 할당
    handleSearch(filteredData);
  };

  return (
    <SearchDiv>
      <StyledInput
        type="text"
        placeholder="검색어를 입력하세요."
        onChange={onChangeKeyword}
        {...rest}
      />
      <SearchBtn onClick={onClickSearch}>
        <SearchIcon />
      </SearchBtn>
    </SearchDiv>
  );
}
```

이렇게 로직을 구성하면 키워드를 입력한 후 검색버튼을 누르면 데이터가 잘 필터링 되는것을 확인할 수 있다.👏🏻👏🏻👏🏻

![인풋창 비웠을때 문제](https://github.com/Side-Effect-Team/side-effect-frontend/assets/71650663/ef6ae8fd-e5d6-436e-89cf-1a9213e160af)

<br />
<br />
<br /><br /><br />
그런데 여기서 문제가 있다.

🧐**_Problem_**

_**1. 검색 버튼 말고 엔터키로 검색할 수 없을까?**_
<br />
_**2. 키워드를 지웠을 때도 필터링된 데이터가 여전히 보인다.**_
<br /><br /><br />

- 1번문제를 해결하기 위해선 위의 StyledInput 태그에 `onKeyDown`속성만 추가해주면 된다.

```typescript
onKeyDown={(e) => {
          if (e.key === "Enter") {
            onClickSearch();
          }
        }}
```

- 2번 문제를 해결하기 위해서는 `onChangeKeyword`함수에 로직을 추가해야한다

```typescript
const onChangeKeyword = () => {
    ...
    if (!e.currentTarget.value) {
      handleSearch(defaultData);
    }
}
```

해당 로직은 검색창이 비워졌을 때, defaultData로 다시 매핑하는 코드이다.
<br /><br /><br />

이제 문제가 해결된 것 같지만 아직 문제는 남아있다🥲
아래의 영상을 보면 컴포넌트를 통해 라우터된 페이지에 방문했다가 뒤로가기를 눌러 페이지에 돌아왔을 때, 키워드가 리셋되어있다..!

![세션문제](https://github.com/Side-Effect-Team/side-effect-frontend/assets/71650663/a05260af-dfc1-42b5-95af-d05370fd91d8)
<br /><br /><br />

🧐**_Problem_**

**_3. 페이지를 다시 방문했을 때 키워드가 리셋되어있는 현상._**

사용자는 필터링 된 데이터에서만을 보기 위해 검색을 했지만 페이지를 방문하고 돌아왔을 때, 키워드가 계속 리셋되어 전체 데이터만을 보여준다면 ux적으로 매우 좋지 못할 것 이다. 이 문제를 해결하기 위해 나는 **세션스토리지에 키워드를 저장**하여 페이지를 다시 방문할 때 저장되어 있는 키워드를 패치시켜 페이지를 보여줄 것 이다.
<br /><br /><br />

- `onClickSearch` 함수가 실행될 때 마다 세션스토리지에 키워드를 저장하는 로직을 추가한다.

```typescript
const onClickSearch = () => {
    ...
    sessionStorage.setItem("searchKeyword", keyword);
}
```

- 그리고 페이지가 리랜더링 될 때 마다 세션에 있는 키워드를 확인해 패치시켜줘야한다. `useState와` `useEffect를` 사용하여 키워드 상태를 관리해주자.

```typescript
const [savedKeyword, setSavedKeyword] = useState("");
useEffect(() => {
  // 이전에 검색한 결과가 있다면 세션스토리지에서 키워드를 꺼내올 수 있다.
  const savedKeyword = window.sessionStorage.getItem("searchKeyword");
  if (savedKeyword) {
    const filteredData = defaultData.filter(
      (data) =>
        data.title.toLowerCase().includes(savedKeyword.toLowerCase()) ||
        data.content.toLowerCase().includes(savedKeyword.toLowerCase())
    );
    handleSearch(filteredData);
    setSavedKeyword(savedKeyword);
  }
}, []);
```

- 여기서 추가로 input값에 defaultValue를 지정해서 사용자가 어떤 키워드를 검색했었는지 알도록 해주자.

```typescript
    <StyledInput
        ...
        defaultValue={savedKeyword}
    />
```

<br /><br /><br />

모든 코드를 적용해주면 짠! 아래와 같이 검색했던 키워드 페이지를 잘 보존하고 있는 것을 볼 수 있다.
여기까지가 오늘의 완성본이다!

![최종2](https://github.com/Side-Effect-Team/side-effect-frontend/assets/71650663/f84b2ce9-66d5-45ff-91fe-48e392ca8b64)

<br /><br /><br />

이렇게 필터 컴포넌트를 구현하며 겪었던 에러와 풀어나갔던 방법을 정리해보았다. 물론 데이터가 많아질수록 대량의 데이터를 하나하나 필터링하는 방법이 썩 좋지는 않겠지만 데이터의 양이 적을 때는 한번 써볼만 한 것 같다.

자 그리고 더 나아가서 굳이 검색버튼, 엔터를 누르지 않아도 키워드를 입력하면 실시간으로 필터링 하는 방식이 있을 것이다. 해당 방식에 대해서도 시행착오를 많이 겪었는데 다음 포스팅에서 정리해보도록 하겠다.😎
<br /><br /><br />
<br /><br /><br />
++ 전체 코드

```typescript
export default function Filter({
  defaultData,
  handleSearch,
  ...rest
}: FilterProps) {
  const [keyword, setKeyword] = useState("");

  const onClickSearch = () => {
    const filteredData = defaultData.filter(
      (data) =>
        data.title.toLowerCase().includes(keyword.toLowerCase()) ||
        data.subTitle.toLowerCase().includes(keyword.toLowerCase())
    );
    sessionStorage.setItem("searchKeyword", keyword);
    handleSearch(filteredData);
  };

  const [savedKeyword, setSavedKeyword] = useState("");

  useEffect(() => {
    const savedKeyword = window.sessionStorage.getItem("searchKeyword");
    if (savedKeyword) {
      const filteredData = defaultData.filter(
        (data) =>
          data.title.toLowerCase().includes(savedKeyword.toLowerCase()) ||
          data.subTitle.toLowerCase().includes(savedKeyword.toLowerCase())
      );
      handleSearch(filteredData);
      setSavedKeyword(savedKeyword);
    }
  }, []);

  const onChangeKeyword = (e: ChangeEvent<HTMLInputElement>) => {
    setKeyword(e.currentTarget.value);
    if (!e.currentTarget.value) {
      handleSearch(defaultData);
    }
  };
  return (
    <SearchDiv>
      <StyledInput
        type="text"
        placeholder="검색어를 입력하세요."
        defaultValue={savedKeyword}
        onChange={onChangeKeyword}
        onKeyDown={(e) => {
          if (e.key === "Enter") {
            onClickSearch();
          }
        }}
        {...rest}
      />
      <SearchBtn onClick={onClickSearch}>
        <SearchIcon />
      </SearchBtn>
    </SearchDiv>
  );
}
```
