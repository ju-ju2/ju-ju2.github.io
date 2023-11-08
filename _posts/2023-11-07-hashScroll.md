---
layout: post
title: hash값에 따라 스크롤 이동하기
subtitle: id와 연결시켜보자
categories: react
tags: [javascript, hash]
image: /assets/images/20231107_hashScroll.png
---

#### 들어가며😃

---

프로젝트 중 하나의 게시물에 트리구조로 자식 게시물이 여러개가 있는 페이지가 있었다. <span style="color: #ff5100;">외부 페이지에서 자식 페이지의 제목을 클릭하면 상세 페이지로 진입 후, 해당 게시물이 있는 영역으로 자동으로 스크롤이 되는 구조</span>를 만들어야 했다. 사실 내가 맡은 파트는 아니였지만 해당 로직이 궁금해서 따로 공부해보았다.
그리고 겪은 실패들과 그 요인을 정리해보려한다.

<br /><br /><br />

한 페이지에서 특정 요소 클릭 시 특정 위치로 스크롤을 이동시키는 방법을 두가지로 정리해보았다.

#### 한 페이지 내에서 스크롤 - href 와 id

---

같은 페이지 내에서는 a 태그의 요소 href에 이동하고 싶은 요소(div)의 id값을 #으로 넣어주면 된다.

```javascript
export default function ATagOnePage() {
  return (
    <>
      <ul>
        <li>
          <a href="#orange">orange</a>
        </li>
        <li>
          <a href="#green">green</a>
        </li>
        <li>
          <a href="#purple">purple</a>
        </li>
      </ul>
      <Orange id="orange">주황</Orange>
      <Green id="green">그린</Green>
      <Purple id="purple">보라</Purple>
      <Pink id="pink">핑크</Pink>
    </>
  );
}
```

<br /><br />

#### 한 페이지 내에서 스크롤 - ref

---

해당 로직은 `scrollIntoView` 메소드를 사용해서 호출된 요소에 스크롤이 이동하게 하는 것이다. 메뉴 버튼에 onClick 이벤트를 할당해주고 컬러박스 요소에 ref로 위치를 참조시켰다.

```javascript
export default function ATagOnePage() {
  const orangeRef = useRef < HTMLDivElement > null;
  const greenRef = useRef < HTMLDivElement > null;
  const purpleRef = useRef < HTMLDivElement > null;

  const onOrangeClick = () => {
    orangeRef.current?.scrollIntoView({ behavior: "smooth" });
  };
  const onGreenClick = () => {
    greenRef.current?.scrollIntoView({ behavior: "smooth" });
  };
  const onPurpleClick = () => {
    purpleRef.current?.scrollIntoView({ behavior: "smooth" });
  };

  return (
    <>
      <div onClick={onOrangeClick}>orange</div>
      <div onClick={onGreenClick}>green</div>
      <div onClick={onPurpleClick} ref={purpleRef}>
        purple
      </div>
      <Orange id="orange" ref={orangeRef}>
        주황
      </Orange>
      <Green id="green" ref={greenRef}>
        그린
      </Green>
      <Purple id="purple" ref={purpleRef}>
        보라
      </Purple>
      <Pink id="pink">핑크</Pink>
    </>
  );
}
```

위의 코드의 예제\_

<img width="500" alt="한 페이지 내에서 스크롤 이동 - href" src="https://github.com/ju-ju2/precamp_class/assets/71650663/73a1dda8-df5c-419b-b740-054ada397874">

<br /><br />

#### 다음 페이지로 넘어가며 스크롤

---

이제 같은 페이지 내에서 스크롤을 이동하는 것이 아니라 다른 페이지로 넘어간 후, 원하는 요소에 스크롤 시켜보자. 우선 코드이다.

```javascript
// 이전 페이지
export default function ATag() {
  const navigate = useNavigate();

  const onClickChangePage = (value: string) => {
    navigate(`changedPage#${value}`);
  };

  return (
    <>
      <div onClick={() => onClickChangePage("orange")}>orange</div>
      <div onClick={() => onClickChangePage("green")}>green</div>
      <div onClick={() => onClickChangePage("purple")}>purple</div>
    </>
  );
}
```

```javascript
// 넘어간 페이지
export default function ATagChangedPage() {
  const location = useLocation();

  useEffect(() => {
    const hash = location.hash.substr(1); // 해당 페이지의 해시값 가져오기 ("#orange"에서 "orange" 등)
    // const hash = location.hash; // #orange
    console.log(hash);
    const element = document.getElementById(hash);

    if (element) {
      element.scrollIntoView({ behavior: "smooth" });
    }
  }, [location.hash]);

  return (
    <>
      <Orange id="orange">주황</Orange>
      <Green id="green">그린</Green>
      <Purple id="purple">보라</Purple>
      <Pink id="pink">핑크</Pink>
    </>
  );
}
```

처음에는 페이지가 잘 넘어가고 해시값까지 주소가 변경이 되는데 요소로 스크롤이 되지않았다. 하루 동안 엄청 답답했는데 아뿔싸,,

<img width="700" alt="한 페이지 내에서 스크롤 이동 - 초기값이 null" src="https://github.com/ju-ju2/precamp_class/assets/71650663/62f2b1a9-82f9-4a2c-ab89-e0a182e66116">

로그를 찍어보니 null 값이 나오는 것을 보고 렌더링 시점에서 요소를 찾지 못한다는 것을 깨달았다. 그리고 위의 코드처럼 호다닥 `useEffect`구문 추가하기,,,
url에 들어오는 hash값을 의존해 해당 값이 변할때마다 리렌더링을 발생시키고 원하는 요소에 스크롤시키는 구문을 만들었다.

완성샷!!

<img width="500" alt="다른 페이지 스크롤 이동" src="https://github.com/ju-ju2/precamp_class/assets/71650663/046095c1-5d2b-4a31-9769-96442efee32b">

<br /><br /><br />

---

구현해보니 굉장히 단순했지만 초반에 실수를 바로 인지못했다는 것이 아쉽다. 그래도 이렇게 구현해보니 꽤 많이 활용되고 잘 응용할 수 있을 것 같아 오늘도 뿌듯함 상승이욤😇
