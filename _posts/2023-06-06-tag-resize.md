---
layout: post
title: 리사이징 될 때마다 변경되는 태그
subtitle: resize가 될떄마다 실행되는 콜백함수
categories: project
tags: [react, resize, callback]
image: /assets/images/2023-06-06-resize/resize.png
---

모집카드 컴포넌트를 제작하면서 발생했던 이슈, 그리고 그것을 해결했던 과정들을 기록하려한다.
우선 아래 컴포넌트 디자인을 살펴보자.
<br />
<br />

- 웹 뷰

<img width="200" alt="스크린샷 2023-06-06 오전 1 21 18" src="https://github.com/Side-Effect-Team/side-effect-frontend/assets/71650663/6609dde9-8ab3-41ee-a293-2318918cc751">
 <br />
 <br />

- 모바일 뷰

<img width="450" alt="태그 디자인" src="https://github.com/Side-Effect-Team/side-effect-frontend/assets/71650663/4c7a7551-b3aa-4071-a106-761e69f8ba9d">

 <br />
 <br />
 모집 게시판에서 글을 작성하면 해당 카드 컴포넌트가 만들어 지는데, 게시글 작성 부분에 모집분야와 사용언어를 선택하는 란이 있다. 해당 태그들은 필터링 기능을 용이하게 하기 위해 이미 정해져 있는 태그로 구성되어 있고, 사용자들이 선택할 수 있는 갯수는 각 6개, 5개로 최댓값이 정해져있다.

그래서 웹 뷰에서는 가장 긴 태그들로만 구성이 되어도 태그가 잘릴 일이 없는 것을 확인했다. 하지만 문제는 모바일 뷰에서 나타났다.

 <br />
 <br />

### Problem 🧐

---

 <br />

<img width="450" alt="태그 잘리는 문제 발생" src="https://github.com/Side-Effect-Team/side-effect-frontend/assets/71650663/7e45cba7-23af-4b0e-894d-82d12c97d435">
 <br />
 <br />

위와 같이 화면을 점점 줄여나가면 태그들이 숨겨진다. 사실 잘! 숨겨지고 있다. `flex-wrap` 속성을 이용해 태그들을 넘치면 아랫줄로 옮겼고 `overflow: hidden` 속성을 통해 아랫줄은 숨겨줬다.
이대로도 문제는 없다만,,,  
 사용자의 입장에서 봤을 때 없어진 태그를 인지 못하고 보이는 부분만 인지할 수 밖에 없다는 점이 매우 마음에 걸렸다.🤦🏻‍♀️

그래서 화면이 리사이징 될 때 마다 숨겨진 태그가 생긴다면,
<em style='font-size: 20px; color: #BAD1C2; font-weight: bold;'>그 태그의 갯수를 오른쪽에 표시해 줄 수 없을까?🧐</em>
라는 대안책을 생각해보았다.
(물론 이는 나를 시험에 며칠간 빠뜨리게했지만,,,)

 <br />
 <br />  
 <br /> 
 <br /> 
 우선 이해를 돕기 위해 바로 결과물을 보자!

### Result 🚀

---

 <br />

<img width="450" alt="태그 숨김 결과" src="https://github.com/Side-Effect-Team/side-effect-frontend/assets/71650663/a24ea64d-898f-4a28-b507-ec36a94f157b">
 <br />
 <br />

그래 바로 이것이다ㅠ 내가 원하는 결과! 태그가 줄어들 때 마다 오른쪽에 숨겨진 태그 수가 실시간으로 반영되는 모습이다!  
(뭔가 태그가 없어질때마다 벽돌깨기 하는 쾌감에 뾰옹~뾰용~ bgm 들리는 건,,, 내가 너무 신난건가?!?)

 <br />
 <br />

물론 바로 이 로직을 구현하진 못했다. 아래에선 이전 코드에서 겪은 어려움과 그 해결과정을 풀어보려한다.

<br />
<br />

### Solution 🔥

---

<br /> 
이 숨겨지는 태그를 어떻게 계산하면 좋을지 생각하고 방법을 알아보다가 태그를 담는 div 요소의 길이와 그 내부에 배치되는 태그의 길이를 계산하는 방식을 택하였다. 아래 코드를 살펴보자

<br />

#### 수정 전 코드

```typescript
export default function TagBox({
  title,
  tagArray,
  fill,
  fontSize,
}: TagArrayProps) {
  const tagRef = useRef<HTMLDivElement>(null); // 1. 태그가 차지할 수 있는 영역
  const [hiddenTagCount, setHiddenTagCount] = useState(0);

  const calculateTagWidths = () => {
    const tagWrapper = tagRef.current;

    if (tagWrapper) {
      const TagWrapperWidth = tagWrapper.offsetWidth;
      const tagsArray = Array.from(tagWrapper.children) as HTMLDivElement[];
      let cumulativeWidth = 0;
      let hiddenCount = 0;
      tagsArray.forEach((tag) => {
        // 2. 각 태그 길이 계산
        const tagWidth = tag.offsetWidth;
        cumulativeWidth += tagWidth + 7; // TagWrapper에 gap이 7px이 할당되어있음

        // 3. 길이가 넘어가는 순간 상태 변화
        if (cumulativeWidth - 7.5 > TagWrapperWidth) {
          hiddenCount++;
          setHiddenTagCount(hiddenCount);
        } else if (cumulativeWidth <= TagWrapperWidth) {
          setHiddenTagCount(0);
        }
      });
    }
  };

  useEffect(() => {
    calculateTagWidths();
  }, [hiddenTagCount]);

  return (
    <TagContainer>
      <TagTitle>{title}</TagTitle>
      <TagWrapper ref={tagRef}>
        {tagArray.map((el, index) => (
          <Tag key={index} fill={fill} fontSize={fontSize}>
            {el}
          </Tag>
        ))}
      </TagWrapper>
      {hiddenTagCount ? (
        <PlusTag fill={fill || "true"}>+ {hiddenTagCount}</PlusTag>
      ) : (
        <></>
      )}
    </TagContainer>
  );
}
```

<br />
<br />
위의 코드의 로직은 이러하다.

<br />

1. 태그가 배치되는 TagWrapper의 길이를 구한다.
2. 각 태그를 `Array.from(tagWrapper.children) as HTMLDivElement[]`로 불러와 각각의 길이를 계산한다. 그리고 태그의 너비를 차례로 더해가며 차지하는 영역을 계산한다. (+7px 은 TagWrapper의 css 속성 중 gap으로 인해 늘어난 부분이 있고, 정확한 계산을 위해 추가로 더해줬다.)
3. 2번에서 축적되어가고 있는 너비값이 TagWrapper의 길이를 넘어가는 순간 숨겨진 태그 hiddenCount의 숫자를 하나씩 높혀간다. 그리고 태그의 갯수가 바뀔 때 마다 setHiddenTagCount를 통해 할당해준다.
   <br />
   <br />
   <br />

아래는 위의 코드의 결과물이다
<br />
<br />

<img width="400" alt="새로고침시에만 태그 수 적용됨" src="https://github.com/Side-Effect-Team/side-effect-frontend/assets/71650663/edf6dbb1-e35c-457b-a5f9-eb9360231229">
<br />
<br />

어랏? 처음에는 숨겨진 태그 갯수가 잘 나온다싶더니! 사이즈를 조정해보았을 때 숨겨진 태그 갯수가 업데이트 되지않고 있다.  
내 머릿속의 로직에선 hiddenCount의 숫자가 올라갈때마다, hiddenTagCount의 상태를 업데이트해주고, hiddenTagCount의 값이 변하면 calculateTagWidths 함수가 실행되면 되는 줄 알았다,,, 하지만 `TagWrapperWidth` 값의 로그를 찍어보니 화면의 크기가 리사이징 되어도 변화가 없었다,, 처음 랜더링 되었을 때의 offsetWidth 값을 계속 유지해서 나머지 요소들도 변함이 없던 것이였다.

<br />
<br /><br />
<br />

이를 해결하기 위해 선택한 방법이 `addEventListener` 메소드의 `resize` 이벤트를 사용하여 브라우저의 창의 크기가 변경될 때 마다 calculateTagWidths 함수를 호출하는 것이다.
코드는 아래 useEffect 부분만 바뀌었다.
<br />
<br />

```typescript
useEffect(() => {
  window.addEventListener("resize", calculateTagWidths);
  return () => {
    window.removeEventListener("resize", calculateTagWidths);
  };
}, []);
```

<br />
<br />
이렇게 바꿔주면 위에서 보았던 것과 같은 결과를 얻을 수 있다.

<br />

<img width="450" alt="태그 숨김 결과" src="https://github.com/Side-Effect-Team/side-effect-frontend/assets/71650663/a24ea64d-898f-4a28-b507-ec36a94f157b">

<br />
사실 resize 이벤트는 브라우저 창의 크기가 변경될 때 많은 이벤트를 발생시키므로 너무 자주 발생할 수 있을 것 같다. 이로 인해 성능 문제가 발생할 수 있지않을까 싶다만, 해당 로직은 모바일에서만 실행되는데 사실 모바일에서 화면을 실시간으로 줄이는 경우는 없지 않은가! 
그런데 또 한편으로는 웹뷰에서도 화면을 줄이면 모바일 뷰가 보이긴 하니 이 문제에 대해 더 생각해봐야겠다. resize 이벤트 핸들러 내에서 타이머 함수를 적용해 문제를 해결할 수 있을 것 같은데,,, 오늘은 이만 여기서 마치고 해결한다면 다시 돌아오겠다😎
