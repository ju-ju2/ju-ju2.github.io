---
layout: post
title: queryString으로 필터 값 설정하기
subtitle: 쿼리스트링으로 api 호출해보자
categories: react
tags: [javascript, search]
image: /assets/images/2023-09-06-queryString/queryString.png
---

#### 들어가며😃

---

입사 후 가장 처음으로 개발한 건이 회원정보를 조회하는 목록페이지였다. 조금 더 풀어보자면, 회원목록이 테이블로 나타나고 필터로 설정된 값을 통해 api를 호출하는 식이였다. 그동안은 state로만 값을 설정하고 api를 호출했었던 터라 처음 개발에 들어가서 조금 버벅였다. 반복되는 목록 페이지 개발에 조금 익숙해진 상태이지만 조금더 개념을 잡을겸, 다음에 참고할겸, 블로그에 개발 로직을 정리해보려한다. 크게 정리하면 쿼리스트링을 받아와 api를 호출하고 쿼리스트링의 값을 아래 이미지의 필터에 넣어주는 형태인데 아래에서 자세하게 알아보자.🧐

<img width="700" alt="필터 레이아웃" src="https://github.com/ju-ju2/ju-ju2.github.io/assets/71650663/18516917-103c-419c-84be-a49507954abc">

<br/><br/>

### 1. 쿼리 파라미터 받아오기

---

페이지에 접근했을 때, 우선 쿼리 파라미터를 받아와야한다. 이번 프로젝트에서는 `query-string` 라이브러리를 통해 파람스를 받아왔다.

- Ex\_ .../search?delivery=GENERAL 의 형태에서 delivery의 값 'GENERAL'을 받아옴

```typescript
import { useLocation } from "react-router-dom";

const { search } = useLocation();

interface queryType {
  category?: string[];
  delivery?: string[];
  keywordType?: "NAME" | "BRAND";
  keyword?: string;
}

const query: queryType = queryString.parse(search, { arrayFormat: "comma" });
```

위에서 `{ arrayFormat: "comma" }` 구문을 추가해주었다. 필터에 들어가는 값이 복수값이 될 경우가 있는데 쿼리스트링에서는 리스트를 어떻게 표현해줄까 하다 콤마(,)를 생각했고, query-string 공식문서에서 콤마로 값을 받아오는 법을 참조하였다.

<br/><br/>

### 2. api 초기 값 설정

---

페이지에 들어오면 어떠한 값을 바탕으로 api를 호출해야한다. 이번 경우엔 쿼리스트링의 값을 보내주어야 하기에 initialParams 라는 변수를 생성하였다.

```javascript
const typescript = {
  category: query.category === ("" || undefined) ? [] : query.category,
  delivery: query.delivery === ("" || undefined) ? [] : query.delivery,
  keywordType:
    query.keywordType === ("" || undefined) ? "NAME" : query.keywordType,
  keyword: query.keyword === ("" || undefined) ? "" : query.keyword,
};
```

페이지에 가장 처음 진입했을 경우엔 당연히 필터링에 설정된 값이 없을 것이다. 그리고 url자체도 `.../search`와 같이 파라미터가 존재하지 않을 수 있다. 그렇기 때문에 위와 같이

1.파라미터 값이 빈값("") 경우 (.../search?category=&delivery),  
 2. 파라미터가 아예 없을 (undefined) 경우 (.../search) 를 고려하여 초기값을 생성하였다.  
 3. 그리고 만약 쿼리파라미터가 들어온다면 (.../search?category='HAIR') `category: query.category`로 값을 설정해주는 것이다.

<br/><br/>

### 3. 검색 조건(필터링) 상태 설정

---

```typescript
const [filter, setFilter] = useState<queryType>({});
```

검색을 위한 필터에 설정된 값을 저장하기 위한 state를 생성한다.

<br/><br/>

### 4. 검색 시, 필터값으로 쿼리스트링 변환

---

```typescript
import { useLocation, useNavigate } from "react-router-dom";

const navigate = useNavigate();

const reloadPage = (params: any) => {
  navigate(
    queryString.stringifyUrl(
      { url: "", query: { ...initParams, ...params } },
      { arrayFormat: "comma" }
    )
  );
};

const onClickSearch = () => {
  reloadPage({ ...filter });
};
```

위의 `reloadPage` 함수는 `useNavigate` 를 사용하여 페이지를 이동시킨다.

- `query: { ...initParams, ...params }` : 기존에 있던 initParams에서 새로 들어온 params값을 재설정하여 쿼리파라미터를 변경한다.
- `{ arrayFormat: "comma" }` : 값은 리스트[]로 저장되지만 쿼리스트링에서는 콤마로 표현한다.

그리고 검색 버튼을 누르면 `onClickSearch` 함수가 실행되고, 이 함수는 state로 저장된 filter 값을 reloadPage 함수의 인자로 할당한다.

⚠️ filter의 state는 특정 selectBox에서 값을 할당할 때 마다 업데이트된다.

<br/><br/>

### 5. 리렌더링 시 쿼리파람스에 맞게 데이터를 다시 불러오기

---

```typescript
useEffect(() => {
  setFilter({});
}, [search]);


// ui
  <Select
    style={{ width: "200px" }}
    mode="multiple"
    allowClear
    placeholder="카테고리 선택"
    options={[
      { value: "HAIR", label: "헤어" },
      { value: "BODY", label: "바디" },
      { value: "HAND", label: "핸드" },
    ]}
    maxTagCount="responsive"
    onChange={(category) => {
      setFilter({ ...filter, category });
    }}
    value={filter.category ?? initParams.category}
  />
</Space>
```

4번 로직에서 reloadPage 이벤트가 일어남에 따라 리렌더링이 발생한다. 사실 위의 구문은 검색하는데 큰 문제가 생기지는 않지만, 뒤로가기가 발생하면 쿼리스트링의 값은 받아오지만 filter의 값은 초기화 되어 값을 제대로 보여주지 못한다. 그렇기 때문에 `useEffect` 를 사용하여 filter를 재설정해주었다. 그리고 `value={filter.category ?? initParams.category}` 해당 부분에서 filter의 값이 비어져 있기때문에 쿼리파라미터 정보가 담겨있는 initParams 의 값을 가져와 필드에 할당하는 것이다.

<br/><br/>

### 6. 결과

---

<img width="700" alt="필터 결과" src="https://github.com/ju-ju2/ju-ju2.github.io/assets/71650663/c9954af2-48bc-40f2-9917-6bc66c23012e">

위와 같이 필터를 설정하고 검색을 하면 url이 변경되고, 뒤로가기가 발생하여도 쿼리스트링의 정보를 가져와 필터들의 값에 잘 할당해주고 있다.

<br/><br/>

---

사실 해당 페이지에는 테이블도 있어야하고, 테이블이 있으면 페이지 사이즈, 현재 페이지와 같은 다른 정보들이 있어야한다. 그리고 그 값들은 변경됨에 따라 바로 쿼리를 변경해줘야하고 로직이 더 복잡해진다. 글이 너무 복잡해질 것 같아서 우선 가장 기본가 되었던 쿼리와 state 관리를 정리해보았다. 이번에 실무을 처음으로 경험해보았고, 참조할만할 자료가 제공되지 않았기때문에 거의 맨땅에 해딩하는 식으로 개발을 진행했다. 해당 목록페이지만해도 개발방식을 3번 갈아엎었는데, 그래도 그 과정에서 다양한 방식을 알게되었고 덕분에 이렇게 정리할 수 있게되었다. 앞으로 이런 로직은 참조자료가 없어도 눈감고 개발할 수 있는 날이 오길..🥹🥹🥹

<br/><br/>

#### <span style="color: #ff5100;">전체코드</span>

---

```typescript
import { Button, Input, Select, Space } from "antd";
import queryString from "query-string";
import React, { useEffect, useState } from "react";
import { useLocation, useNavigate } from "react-router-dom";

interface SearchProps {}
interface queryType {
  category?: string[];
  delivery?: string[];
  keywordType?: "NAME" | "BRAND";
  keyword?: string;
}

const SearchPage = ({}: SearchProps): React.ReactElement => {
  const navigate = useNavigate();
  const { search } = useLocation();

  // 1. 상단에 쿼리스트링 파람스를 받는다.
  const query: queryType = queryString.parse(search, { arrayFormat: "comma" });

  // 데이터를 보내줄 상태를 선언한다
  const initParams = {
    category: query.category === ("" || undefined) ? [] : query.category,
    delivery: query.delivery === ("" || undefined) ? [] : query.delivery,
    keywordType:
      query.keywordType === ("" || undefined) ? "NAME" : query.keywordType,
    keyword: query.keyword === ("" || undefined) ? "" : query.keyword,
  };

  // 2. 검색 조건 상태를 저장할 state를 생성한다.
  const [filter, setFilter] = useState<queryType>({});

  // 3. 필터로 페이지를 이동한다.
  const onClickSearch = () => {
    reloadPage({});
  };

  // 4. 페이지가 변할때 마다 값을 업데이트 시킨다
  useEffect(() => {
    setFilter({});
    // 쿼리파람스에 맞게 데이터를 다시 불러온다.
  }, [search]);

  // 5. reload를 통해 쿼리를 변경하고 위의 useState를 통해 값을 업데이트시킨다.
  const reloadPage = (params: any) => {
    navigate(
      queryString.stringifyUrl(
        { url: "", query: { ...initParams, ...params } },
        { arrayFormat: "comma" }
      )
    );
  };

  // 리셋
  const onClickReset = () => {
    setFilter({});
    reloadPage({
      category: "",
      delivery: "",
      keywordType: "NAME",
      keyword: "",
    });
  };

  return (
    <div style={{ width: "800px" }}>
      <div style={{ width: "100%" }}>
        <Space direction="vertical" style={{ marginRight: "10px" }}>
          <div>카테고리</div>
          <Select
            style={{ width: "200px" }}
            mode="multiple"
            allowClear
            placeholder="카테고리 선택"
            options={[
              { value: "HAIR", label: "헤어" },
              { value: "BODY", label: "바디" },
              { value: "HAND", label: "핸드" },
            ]}
            maxTagCount="responsive"
            onChange={(category) => {
              setFilter({ ...filter, category });
            }}
            value={filter.category ?? initParams.category}
          />
        </Space>
        <Space direction="vertical" style={{ marginRight: "10px" }}>
          <div>배송</div>
          <Select
            style={{ width: "200px" }}
            mode="multiple"
            allowClear
            placeholder="배송 선택"
            options={[
              { value: "GENERAL", label: "일반 배송" },
              { value: "FREE", label: "무료 배송" },
            ]}
            maxTagCount="responsive"
            onChange={(delivery) => setFilter({ ...filter, delivery })}
            value={filter.delivery ?? initParams.delivery}
          />
        </Space>
        <Space direction="vertical">
          <div>검색</div>
          <div style={{ display: "flex", gap: "10px" }}>
            <Select
              style={{ width: "100px" }}
              value={filter.keywordType ?? initParams.keywordType}
              options={[
                { value: "NAME", label: "제품명" },
                { value: "BRAND", label: "브랜드" },
              ]}
              onChange={(keywordType) => setFilter({ ...filter, keywordType })}
            />
            <Input
              placeholder="검색어를 입력해주세요"
              onChange={(e) =>
                setFilter({ ...filter, keyword: e.target.value })
              }
              value={filter.keyword ?? initParams.keyword}
              style={{ width: "270px" }}
            />
          </div>
        </Space>
      </div>
      <div
        style={{
          marginTop: "10px",
          display: "flex",
          gap: "10px",
          justifyContent: "flex-end",
        }}
      >
        <Button onClick={onClickReset}>초기화</Button>
        <Button type="primary" onClick={onClickSearch}>
          검색
        </Button>
      </div>
    </div>
  );
};

export default SearchPage;
```
