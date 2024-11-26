---
layout: post
title: React와 Next.js의 Rendering 방식
subtitle: csr,ssr만 있는게 아니라구~
categories: Next
tags: [Next.js, rendering]
image: /assets/images/20241126_next_rendering.png
---

### 😵 들어가며

---

리액트와 넥스트의 차이점이라 하믄 렌더링을 어디서 하느냐며, CSR과 SSR 의 차이라고만 생각했다. 하지만 SSR에도 종류가 여러가지가 있었는데,,! 정확히는 Next.js의 렌더링 종류엔 여러가지가 있는데,,!
를 깨닫고 써보는 오늘의 포스팅이다.

<br/>
<br/>

### 🔥 기본 개념 정리

---

**React의 Rendering**

리액트에서는 브라우저가 서버에서 JavaScript, CSS, HTML 파일을 다운로드한 후, JavaScript가 실행되어 화면을 렌더링한다. 이 과정에서 사용자는 초기에 빈 화면을 보게 되며, 초기 빈 껍데기인 html로 인해 검색엔진최적화(SEO)가 불리하다.

<br/>

**Next.js의 Pre-rendering**

페이지의 HTML을 미리 생성하는 기능으로, 사용자가 요청하기 전에 서버나 빌드 단계에서 HTML을 준비해 두는 것을 뜻한다. 그로인해 이미 준비된 HTML로 사용자는 기다림 없이 화면을 볼 수 있고, SEO가 개선된다.

<br/>

**Hydration**

Next.js에서 Pre-rendering은 HTML과 CSS만을 준비해둔 상태로 빈 껍데기에 불과하다. 브라우저에서 JavaScript가 실행되기 전까지는 페이지와 사용자가 상호작용할 수 없다. 예를 들어, 버튼과 같은 컴포넌트는 초기에는 동작하지 않는 것이다. 브라우저가 JavaScript 코드를 실행하여 React 컴포넌트와 HTML을 연결하고, 동작을 활성화하는 과정이 진행되는데, 이를 **Hydration**이라고 한다.

<br/>
<br/>

### 🌸 Next.js의 Rendering

---

위의 개념에서 정리했던 것 처럼 클라이언트(1)와 서버단(3)에서 렌더링 할 수 있다. 분류를 하면 아래의 총 4가지의 렌더링 방식이 존재한다. 이는 Next.js 13 버전 기준의 방식이다.

**CSR**: Client Side Rendering

- 클라이언트에서 직접 데이터를 fetch하는 방식
- 코드 상단에 'use client' 구문을 추가하면 기존 리액트 방식처럼 CSR 방식을 구현할 수 있다.
- 아래 렌더링 결과는 렌더링시의 시간을 그대로 출력한다. 새로고침 마다 시간이 변경되어 데이터를 계속 요청하는 것을 확인할 수 있다.

<br/>

```typescript
"use client";

import { useEffect, useState } from "react";

export default function CSRPage() {
  const [time, setTime] = useState<string>("");

  useEffect(() => {
    const getTime = async () => {
      const res = await fetch(
        "http://worldtimeapi.org/api/timezone/Asia/Seoul",

        {
          cache: "no-store",
        }
      );

      const data = await res.json();

      setTime(data.datetime);
    };

    getTime();
  }, []);

  return <div>CSR: {time}</div>;
}
```

<br/>

<img width="300" alt="csr" src="https://github.com/user-attachments/assets/44dcd38a-5e9c-43c8-89a3-634145b45527">

<br/>
<br/>

**SSR**: Server Side Rendering

- 사용자가 요청했을 때, 서버에서 바로 생성하여 제공된다.
- 요청 마다 다른 데이터가 보여지는 페이지에 적합한 렌더링 방식 이다.
- SSR 방식 역시, 렌더링을 요청한 시점의 시간을 출력하며 매번 새로운 데이터 호출로 시간이 변경된다.

<br/>

```typescript
export default async function SSRPage() {
  const res = await fetch("http://worldtimeapi.org/api/timezone/Asia/Seoul", {
    cache: "no-store",
  });

  const data = await res.json();

  return <div>SSR: {data.datetime}</div>;
}
```

<br/>

<img width="300" alt="ssr" src="https://github.com/user-attachments/assets/20395316-6eda-442a-9b52-ebcad4085542">

<br/>
<br/>

**SSG**: Static Site Generation

- 페이지가 빌드 시점에 미리 생성되어 서버에 제공될때, 미리 만들어둔 HTML을 재사용하는 렌더링 방식이다.
- 변경되지 않는 콘텐츠에 적합하다.
- 해당 이미지를 참고하면, 빌드 시에 저장된 시간으로만 계속 출력하여 업데이트하지 않는 것을 확인할 수 있다.

<br/>

```typescript
export default async function SSGPage() {
  const res = await fetch("https://worldtimeapi.org/api/timezone/Asia/Seoul");

  const data = await res.json();

  return <div>SSG: {data.datetime}</div>;
}
```

<img width="300" alt="ssg" src="https://github.com/user-attachments/assets/cfd21fd2-940e-4a0a-bd57-679d72d0c002">

<br/>
<br/>

**ISR**: Incremental Static Regeneration

- SSG와 SSR의 중간 형태로, 페이지는 빌드 시 생성되지만 특정 시간마다 서버에서 다시 생성하여 주기적으로 업데이트가 가능하다.
- 코드에서 10초마다 갱신하는 업데이트 주기를 옵션으로 추가하였는데, 새로고침 시 변동하는 시간을 확인할 수 있다.

<br/>

```typescript
export default async function ISRPage() {
  const res = await fetch("http://worldtimeapi.org/api/timezone/Asia/Seoul", {
    next: { revalidate: 10 },
  }); // 10초마다 갱신

  const data = await res.json();

  return <div>ISR: {data.datetime}</div>;
}
```

<img width="300" alt="isr" src="https://github.com/user-attachments/assets/f6963716-d5f5-48a8-981f-db407ef01ee4">

<br/>
<br/>

### 😀 마무리

---

CSR 자체는 클라이언트 단에서 대부분의 작업을 수행하기 때문에 서버가 하는 역할이 거의 없다. 반면에 SSR은 서버에서 계속 새로운 HTML을 만들어내야 하기 때문에 상대적으로 부하가 올 수 밖에 없다. 그렇기 떄문에 사용자에게 보여줄 화면에 따라 다른 렌더링 방식을 채택하는 것은 사용자 경험과 성능 측면에서 매우 이점이 있다.
왜 이걸 지금에야 정리했을까 반성 + 오늘 정리한 개념을 바탕으로 더 나은 사이트를 만들 수 있을 것 같은 자신감과 함께,,,넥스트를 더 잘 이해한 듯 한 느낌을 가져가며,, 오늘의 포스팅 마무리해보겠다.🤗
