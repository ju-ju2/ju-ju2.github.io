---
layout: post
title: 테이블에서 체크박스 선택시 row 클릭이 되는 현상 방지
subtitle: antd 라이브러리 파해쳐버려
categories: trouble_shooting
tags: [antd, 트러블슈팅]
image: /assets/images/20240303_trouble_shooting_rowClick.png
---

#### 들어가며😃

---

지난 프로젝트를 생각해보면 테이블에 대한 이슈가 꽤 나왔다. 다음 프로젝트에서도 꽤나 비슷한 상황들이 나오길래 정리해두면 좋겠다 싶어 정리해보는 내용이다. 이번 포스팅은 테이블의 체크 박스 선택 시 일어나는 에로 사항과 그 해결 과정이다.

<br/><br/>

#### 이슈 재현😵‍💫

---

![2024-03-035 12 53-ezgif com-video-to-gif-converter](https://github.com/ju-ju2/trouble_shooting/assets/71650663/a10f40de-4594-48e5-ab97-3f814ada34a6)

이미지에서 보는 것 처럼, 테이블의 row 를 클릭하면 해당 데이터에 대한 상세페이지로 접근한다.

![2024-03-035 12 24-ezgif com-video-to-gif-converter](https://github.com/ju-ju2/trouble_shooting/assets/71650663/0b1c5b0e-af3c-452b-a4e8-64255a47e0d5)

그런데,,,,!  
체크박스를 선택하다보면 해당 영역 그 이외의 부분을 클릭하는 경우가 있다. 빠르게 리스트를 체크하거나 대충(?) 체크하다보면 다른 영역을 선택하는 경우가 허다했다. 그러면 다시 테이블 페이지로 돌아갔을때, 하나하나 선택했던 체크 항목들이 해제되어 있는 모습에 속상해진다,,  
사실 해당 이슈는 그렇게 크리티컬한 이슈는 아니다. qa에서도 딱히 잡아내진 않았지만 사용하는 입장에서 좀 불편할 것 같아 해결책을 찾아보았다.

<br/><br/>

#### 에러 처리 방법🚀

---

해결과정이 순탄하지는 않았다. antd 라이브러리를 찾아봐도 체크박스 영역을 제어하는 api는 없었다.
체크박스는 rowSelection 속성을 추가하면 자동적으로 생기게 되고, 테이블의 행을 클릭하는 onClick 속성은 onRow라는 속성 내부에 있다. 서로가 서로를 제어하는 속성은 찾을 수 없어 내가 선택한 방법은 css 제어였다.

<br/>

<img width="700" alt="개발자 모드 css" src="https://github.com/ju-ju2/trouble_shooting/assets/71650663/5cbf2e8d-35c8-44fe-a0a9-03b879301571">

테이블의 체크포인트 영역은 `ant-table-cell ant-table-selection-column` 이라는 className이 지정되어있다. `ant-table-cell`은 각 컬럼마다 모두 지정되어 있는 className이였고, `ant-table-selection-column` 는 체크 박스가 생기는 영역에만 생성되는 className으로 해당 부분을 제어하면 되겠다는 생각이 들었다.

<br/>
클릭 이벤트 발생 시 해당 className을 가지고 있다면 onClick 이벤트를 막아주는 로직을 구현해보았다.

```typescript
<Table
  rowSelection={{
    ...rowSelection,
  }}
  columns={columns}
  dataSource={data}
  onRow={() => {
    return {
      // 체크박스 열 클릭 시 이벤트 막음
      onClick: (e) => {
        const target = e.target as HTMLElement;
        if (target.classList.contains("ant-table-selection-column")) {
          return;
        } else {
          navigate("/~~");
        }
      },
    };
  }}
/>
```

<br/>

해당 onClick 이벤트시 들어오는 인자를 log로 찍어보면 아래와 같다.

<img width="600" alt="image" src="https://github.com/ju-ju2/trouble_shooting/assets/71650663/30743797-60e8-4ee7-84a5-1ad2ab841618">

그래서 코드에서 `const target = e.target as HTMLElement;` 이 부분을 통해 이벤트 객체(e)에서 실제 클릭된 요소(target)를 가져와서 타입을 HTMLElement로 변환해준다. 그럼 아래와 같이 변환된다.

<img width="204" alt="image" src="https://github.com/ju-ju2/trouble_shooting/assets/71650663/e9cce2cf-6304-4131-8dde-149b91cfea9b">

그리고 이제 특정한 className을 가진 요소에 대한 제어를 해주면!!

![2024-03-035 13 51-ezgif com-video-to-gif-converter](https://github.com/ju-ju2/trouble_shooting/assets/71650663/ea1d16b3-da7d-4c24-ab47-42e0dc64a5ed)

결과는 짜잔~😎  
체크 셀에 있는 영역을 클릭해도 더이상 디테일 화면으로 이동하지 않는다. 미션 클리어~

<br/><br/><br/><br/>

---

사실 처음 프로젝트를 진행할 때는 이 이슈를 해결하진 못하였다. 시간이 너무 부족하기도 하고 이슈라고 하기에도 애매한 이슈라 그 당시에는 넘어갔었다. 그런데 이번 프로젝트를 진행하면서 똑같은 라이브러리를 쓸테니 똑같은 이슈가 나올 것이라 예상했다. 그래서 혼자 이것저것 건드려보니 해결법을 찾아냈고 동료들에게 해당 로직을 공유할 수 있을 것 같다.ㅎㅎ 뿌듯하구만 🐥🐓🆙
