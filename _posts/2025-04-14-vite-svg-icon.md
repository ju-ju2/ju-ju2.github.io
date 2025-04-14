---
layout: post
title: vite 환경에서 Icon 컴포넌트 만들기
subtitle: vite svg 설정
categories: Vite
tags: [Vite]
image: /assets/images/20250414_vite_icon.png
---

### 😵 들어가며

---

지난 게시글에서 vite 환경에서 svg를 리액트 노드 컴포넌트로 가져오는 법을 포스팅했었다. 그리고 오늘은 이어서, 가지고 온 컴포넌트를 Icon 컴포넌트 하나로 관리한 코드를 공유해보려 한다.

<br/>
<br/>

### 전체 코드

---

```
import Delete from '../icons/svg/delete.svg?react';
import Loading from '../icons/svg/loading.svg?react';
import Note from '../icons/svg/note.svg?react';
import Seed from '../icons/svg/seed.svg?react';
import Down from '../icons/svg/triangle-down.svg?react';
import Up from '../icons/svg/triangle-up.svg?react';

export const icons = {
   Loading,
   Delete,
   Note,
   Up,
   Down,
   Seed,
};

export type IconNameType = keyof typeof icons;

interface IconProps {
   size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl' | 'xxl';
   color?: 'primary' | 'secondary' | 'danger' | 'white';
   name: IconNameType;
   onClick?: (e: React.MouseEvent<HTMLDivElement>) => void;
}

const Icon = ({ size = 'sm', color = 'primary', name, onClick }: IconProps) => {

const IconElement = icons[name];

const sizeMap = {
   xs: 12,
   sm: 16,
   md: 24,
   lg: 32,
   xl: 48,
   xxl: 60,
};

const colorMap = {
   primary: '#1677FF',
   secondary: '#bfbfbf',
   danger: '#ff4d4f',
   white: '#ffffff',
};

return (
   <div
      style={{
      width: sizeMap[size],
      height: sizeMap[size],
      display: 'inline-block',
      }}
      onClick={onClick}>
      <IconElement fill={colorMap[color]} />
   </div>
   );
};

export default Icon;
```

<br/>
<br/>

### 구조 설정

---

위의 코드를 보면 알 수 있듯, div 라는 부모 태그 내부에 icon 태그로 구성하였다. 그리고 부모 태그에 스타일을 주었는데 해당 사이즈와 컬러를 자식인 svg 가 받을 수 있도록 svg 파일을 수정해주어야한다. 아래 svg 파일에서 width, height을 각 100%로 수정하고 색상이 반영되어야할 부분을 current 혹은 currentColor로 변경해준다.

<br/>

```
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 200 200" width="100%" height="100%">
<circle fill="current" stroke="current" stroke-width="15" r="15" cx="40" cy="100">
<animate attributeName="opacity" calcMode="spline" dur="2" values="1;0;1;" keySplines=".5 0 .5 1;.5 0 .5 1" repeatCount="indefinite" begin="-.4"></animate>
</circle>
</svg>
```

<br/>
<br/>

### icon 자식 요소 갈아끼우기

---

아이콘을 하나하나 만드는 것 이라면 `<div><Loading /></div>` 식으로 썻을텐데, 나는 현재의 Loading 컴포넌트를 as 패턴 마냥 이름만 받아 갈아치우게 할 것이다.

```
import Delete from '../icons/svg/delete.svg?react';
import Loading from '../icons/svg/loading.svg?react';

export const icons = {
   Loading,
   Delete,
};

export type IconNameType = keyof typeof icons;
```

불러온 svg 컴포넌트를 우선 icons 라는 객체에 담는다. 그리고 중요한건 바로 이부분 `export type IconNameType = keyof typeof icons;`
keyof typeof를 사용하면 객체의 키를 타입으로 추출할 수 있다.

따로 또 같이 뜯어보자.

- typeof : 각 키에 할당된 값의 타입을 가져온다.

```
const user = {
  id: 1,
  name: 'Alice',
  age: 25,
};

type UserType = typeof user;

// UserType = { id: number; name: string; age: number; }
```

- keyof: 객체 타입의 키를 문자열 타입으로 반환한다.

```
type UserKeys = keyof UserType;
// UserKeys = 'id' | 'name' | 'age'
```

- keyof typeof: 합쳐서 객체의 키를 문자열 타입으로 반환한다.

```
const user = { id: 1, name: 'Alice', age: 25, }; type UserKeys = keyof typeof user;
// UserKeys = 'id' | 'name' | 'age'
```

이러한 속성을 이용해 Import 한 svg 들을 아이콘을 객체에 담고 타입으로 만들어주었다.

그리고 사용시에는 객체의 키를 가져오는 코드를 사용한다.

```
export type IconNameType = keyof typeof icons;
const Icon = (name: IconNameType) => {

// 객체의 키를 가져옴
const IconElement = icons[name];

return (
   <div>
      <IconElement/>
   </div>
   );
};
```

이렇게 하면 name에 따라 내부 자식 컴포넌트가 바뀌는 구조가 된다!

<br/>
<br/>

### 마무리

---

해당 문법을 사용하면서 키값을 타입으로 하기 때문에 굳이 다시 타입을 만들어줄 필요가 없었고, 잘못된 값이 사용될 가능성을 배제할 수 있었다. 또한 가독성을 높이는데 유용했고 나의 프로젝트 중 생각보다 많은 코드에 썼던 걸 확인했다. 잘 기억한 후 다양한 곳에 많이 활용할 수 있도록 기록해보는 오늘의 포스팅,,,끝~
