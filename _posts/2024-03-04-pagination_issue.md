---
layout: post
title: 테이블 페이지네이션시 선택된 데이터 사라지는 이슈
subtitle: antd 라이브러리 파해쳐버려
categories: trouble_shooting
tags: [antd, 트러블슈팅]
image: /assets/images/20240304_trouble_shooting_pagination.png
---

#### 들어가며😃

---

프로젝트에서 Antd 테이블을 사용하다가 문제를 발견했다. 페이지네이션 중 체크박스를 선택하고 다른 페이지에서 체크를 하고 다시 돌아오면 이전 페이지의 체크가 해제되어있었다. 간략하게 정리하자면 antd 테이블의 rowSelection 속성이 페이지별로 체크된 데이터를 저장하고 있는데 실제 api는 전체데이터가 아닌 페이지 별로 데이터를 받아오기 때문에 저장이 되지 않는 이슈였다. 해당 이슈를 어떻게 해결해나갔는지 정리해보고자 올리는 오늘의 포스팅이다.

<br/><br/>

#### 이슈 재현

---

\*해당 이슈를 재현하기 위해 faker 라이브러리를 사용하였고, 데이터 생성기는 [해당 페이지](https://ju-ju2.github.io/faker/2024/03/04/faker.seed.html){: target="\_blank" }에서 참고 가능하다!

![2024-03-049 54 40-ezgif com-video-to-gif-converter](https://github.com/ju-ju2/trouble_shooting/assets/71650663/5047ed74-a27c-4e53-94ae-f6eed4930244)

위에서 언급했던 이슈처럼 체크를 하면 이전 페이지에서 체크한 항목이 선택 해제가 되는 모습을 보게 된다. antd 페이지에서는 이런게 전혀 없었는데,,! 당황스럽다..!😵‍💫 어떤 차이가 있는지 확인하기 위해 로그를 찍어보았다.

[Antd 공식 홈페이지](https://ant.design/components/table){: target="\_blank" } 에서 페이지네이션을 보면 50개 정도의 대량의 데이터를 한번에 내려주고 있다. 그리고 체크를 하면 체크된 항목들이 배열에 저장되는 모습을 볼 수 있다.

![2024-03-0510 54 04-ezgif com-video-to-gif-converter](https://github.com/ju-ju2/ju-ju2.github.io/assets/71650663/508a957d-3fe8-4e59-870f-69e7e45e6f6f)
<br/><br/>

그런데,,! 프로젝트에서 사용하는 테이블에서는 페이지를 넘겨 선택을 하면 기존에 저장되었던 선택 항목은 사라진다. 딱 내가 머물러 있는 페이지만 키가 저장되는 것,,!

![2024-03-0510 54 45-ezgif com-video-to-gif-converter](https://github.com/ju-ju2/ju-ju2.github.io/assets/71650663/c5f24ac9-4231-464e-81b4-32b68ee970c6)

이것저것 시도해보다가 깨닫게 된 원인은 바로 <span style="color: #ff5100;">우리는 **실제 api**를 쓰고 있기 때문이였다.</span> 대게 비슷한 방식으로 api가 구현될텐데, 테이블에 얼마나 많은 데이터가 들어갈지 모르고 그 많은 데이터를 한번에 호출한다면 속도가 나지 않을 것이다. 그래서 페이지 별, 요청한 데이터의 갯수, 총 데이터 갯수를 파라미터로 api를 요청하고 서버쪽에서는 우리가 필요한 데이터를 끊어서 보내주게 된다.  
그래서 체크된 정보까지 다 날아가는 것이였다,,!

<br/>

이를 해결하기 위해서는 <u>페이지 호출에 상관없이 선택한 모든 키를 따로 저장하는 로직을 추가로 구현해야했다.</u>

<br/><br/>

#### 해결 과정 1: 모든 키 저장

---

```typescript

  const [selectedRowKeys, setSelectedRowKeys] = useState<Key[]>([]);

    // <Table
        rowSelection={
          selectedRowKeys,
          onSelect: (record) => {
              setSelectedRowKeys((prev) => [...prev, record.key])
          },
        ...
        }
```

우선 테이블의 체크 액션에 할당되는 `onSelect` 속성에서 row 정보인 `record를` 받아온다. 그리고 state에 스프레드 연산자를 통해 체크되는 항목을 모두 저장해준다. 그런데 문제는!  
체크된 항목을 다시 선택하면 아래 이미지와 같이 키가 중복으로 저장이 되며 선택 해제가 되지 않는다.

![2024-03-1510 21 54-ezgif com-video-to-gif-converter](https://github.com/ju-ju2/ju-ju2.github.io/assets/71650663/a33970a9-5688-4104-bae6-10e84e95b98d)

그래서 <u>선택된 키가 이미 저장되어있는 키인지 아닌지를 파악</u>해야 할 필요가 있다.

<br/>

#### 해결 과정 2: 중복 키 제외

---

```typescript

  const [selectedRowKeys, setSelectedRowKeys] = useState<Key[]>([]);

      // <Table
        rowSelection={
          selectedRowKeys,
          onSelect: (record) => {
            // 이미 존재하는 키인지 아닌지 구분
            if (selectedRowKeys.includes(record.key)) {
              setSelectedRowKeys(
                selectedRowKeys.filter((key) => key !== record.key)
              );
            } else {
              setSelectedRowKeys((prev) => [...prev, record.key]);
            }
          },
        ...
        }

```

위의 수정된 코드에서는 이미 저장된 키라면 `filter`를 통해 제외시키고 있다.

![2024-03-1510 24 02-ezgif com-video-to-gif-converter](https://github.com/ju-ju2/ju-ju2.github.io/assets/71650663/ab1a68ed-bf6f-46cf-94fd-ade54fceb096)

그리고 짜잔! 1차 완성샷이다.

<br/>

#### 해결 과정 3: 전체선택 로직 구현

---

아직 할 일이 남았다.

![2024-03-1511 07 02-ezgif com-video-to-gif-converter](https://github.com/ju-ju2/ju-ju2.github.io/assets/71650663/24d3a5f8-5bb0-4883-a726-cdbe299fdc66)

위의 이미지와 같이 전체선택 로직이 안먹히기 때문에 따로 `onSelectedAll` 이라는 속성을 정의해 줄 필요가 있다.

<br/>

```typescript
      // <Table
        rowSelection={
        ...
          onSelectAll: (isSelected, _selectedRows, changeRows) => {
            // isSelected 전체 선택 모드 boolean
            // changeRows : 이미 선택된 체크 항목 이외의 체크 로우들 정보
            if (isSelected) {
              setSelectedRowKeys((prev) => [
                ...prev,
                ...changeRows.map((item) => item.key),
              ]);
            } else {
              setSelectedRowKeys(
                selectedRowKeys.filter(
                  (key) => !changeRows.map((item) => item.key).includes(key)
                )
              );
            }
          },
        }
        ...
```

`onSelectedAll` 에서 나는 `isSelected` 와 `changeRows` 인자를 사용하였다.
**isSelected** 는 현재 페이지가 전체 선택 상태인지 아닌지를 불린값으로 보여주고 테이블에 5개의 항목이 있고, 5개 항목이 모두 선택되면 true 값을 반환한다.  
**changeRows** 는 전체 선택을 했을때, 이미 선택된 항목을 제외하고 나머지 로우 정보를 전달해준다.  
그래서 나는 위와 같이 로직을 구현하였다.

<br/><br/>
그리고 최종 결과! 짠🏋🏻

![2024-03-1511 14 29-ezgif com-video-to-gif-converter](https://github.com/ju-ju2/ju-ju2.github.io/assets/71650663/5b4fe233-ae9e-4421-a612-a33827eaa812)

<br/><br/><br/><br/>

---

사실 원인을 파악하는데도 쉽지는 않았다...나는 쌩 초짜였기 때문이지,,,🐥 api가 어떻게 구현되어있지? 어랏? 10개씩 주네? 부터 시작된 깨달음이 문제 해결의 실마리가 되고 실제로 이슈를 해결해보면서 이슈가 발생되었을때는 무조건 내가 짠 로직에서만 문제를 찾을 것이 아니라 api에서는 어떤 값을 주고 있는지, 연관 관계가 어떻게 되는지 등 더 다양하게 관점을 가지고 문제를 바라볼 필요가 있음을 많이 느꼈다. 나는 신입이니까 무조건 내실수일꺼야! 하며 주구장항 코드만 보고있던 시간이 아주 야속했지만 좋은 깨달음의 시간이였다. 🧐
<br/><br/>
+++ 추가  
이 이슈를 실무에서 해결할 당시엔 꽤 복잡하게 구성했던 기억이 있는데 지금 와서 정리하니 더 단순하게 구성할 수 있었다. 나,,,실력이 좀 는건가,,,!😵‍💫 이렇게 쉬운 구조를 그땐 왜 그렇게 헤맸는지ㅠ 다음 이슈는 더 잘 헤쳐나가길 바라며!🚀
