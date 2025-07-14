---
layout: post
title: Figma 플러그인 메세지 통신
subtitle: 자동화를 위한 멀고도 험한 길
categories: Figma
tags: [Vite, Figma]
image: /assets/images/20250710_figma_plugin.png
---

### 😵 들어가며

---

<img src="/assets/images/2025-07-14-figma-plugin/figma-plugin.png"/>.

짜잔 내가 만든 플러그인

<br />

이번엔 꽤 흥미로웠던 피그마 UI 단과 CODE 와의 상호 통신 방식에 대한 글이다. 하긴 분리되어있는 환경에서 어떻게 통신하는 거지 했는데, 메시지를 전달하면서 트리거를 주는 것이였다. 자세한 내용은 아래에서 참고해보쟝

<br />
<br />

### 📦 Figma Plugin에서의 메시지 통신 정리

---

Figma 플러그인은 UI와 Code가 분리되어 있으며, 직접적인 함수 호출이 불가능하다. 따라서 양측은 postMessage 방식을 통해 메시지를 주고받아야 하며, 이를 통해 상태 전달 및 이벤트 트리거가 이루어지게 된다.

<br />
<br />

### ✅ figma.ui.postMessage

---

Code에서 UI로 메시지를 전달할 때 사용하는 기본 내장 API.

```
figma.ui.postMessage({ type: FIGMA_MESSAGE.LOADING_START });
figma.ui.postMessage({ type: FIGMA_MESSAGE.LOADING_END });
```

예를 들어, UI에서 버튼을 눌렀을 때 Code 측에서 API 요청을 한다고 가정하면, UI는 API 통신 상태를 직접 알 수 없습니다. 이때 figma.ui.postMessage를 통해 "지금 API 호출 중이야~" 같은 메시지를 보내주면, UI에서는 로딩 상태나 버튼 비활성화 등 사용자에게 상태를 반영할 수 있습니다.

<br />
<br />

### ✅ window.addEventListener와 pluginMessage 수신

---

UI 측에서는 pluginMessage를 통해 Code로부터의 메시지를 수신한다.

```
useEffect(() => {
const messageHandler = (e: MessageEvent) => {
const message = e.data.pluginMessage;

    	if (message.type === FIGMA_MESSAGE.LOADING_START) {
    		setIsLoading(true);
    	}
    	if (message.type === FIGMA_MESSAGE.LOADING_END) {
    		setIsLoading(false);
    	}
    };

    window.addEventListener("message", messageHandler);
    return () => {
    	window.removeEventListener("message", messageHandler);
    };

}, []);
```

나는 처음에 메시지를 언제 보내는지도 모르는데 useEffect 의존성 배열에 뭐라도 넣어야 되는거 아닌가 싶었는데 한번만 등록하면 계속 메시지를 들을 수 있다고 한다.

💡 의존성 배열을 비워도 괜찮은 이유?
window.addEventListener는 `전역 객체`에 이벤트 리스너를 등록하므로, useEffect 내에서 한 번만 등록해도 메시지를 계속 수신할 수 있다.

<br/>
<br/>

### ✅ @create-figma-plugin/utilities의 emit과 on

---

Figma 플러그인 라이브러리인 @create-figma-plugin/utilities는 위의 메시지 통신 과정을 더 깔끔하고 간결하게 도와준다.

- `emit` – 메시지 발신

```
import { emit } from "@create-figma-plugin/utilities";

emit("MY_EVENT", { foo: "bar" });
```

내부적으로는 window.postMessage를 사용하고 코드 간 복잡한 메시지 포맷을 추상화하여 보여준다.
`code.ts → ui.tsx` 또는 그 반대로 메시지를 보낼 수 있다.

<br/>
<br/>

- `on` – 메시지 수신

```
import { on } from "@create-figma-plugin/utilities";

on("MY_EVENT", (payload) => {
  console.log("받은 값:", payload);
});
```

이벤트 이름에 맞춰 콜백 함수를 등록한다.
`ui.ts → code.tsx` 또는 그 반대로 메시지를 받을 수 있다.

<br/>
<br/>

### ✅ 타입 가드를 통한 안전한 메시지 통신

---

나는 조금 더 안전한 타입을 사용하기 위해 emit, on 을 한번 감싸주었다. 특정한 이름을 가진 메세지로 데이터를 주고받을때, 주는 쪽과 받는 쪽의 데이터 양식이 다르지 않도록 하기 위해서다. 이렇게 하면 오타나 잘못된 타입 전달을 방지할 수 있어, 유지보수와 협업에서 안정성이 크게 향상될 수 있다!

```
import { emit as e, on as o } from "@create-figma-plugin/utilities";

export type Events = {
  PULL_REQUEST_STYLES: {
    name: typeof FIGMA_EVENT.PULL_REQUEST_STYLES;
    payload: GithubPayload;
    handler: (props: GithubPayload) => void;
  };
};

type EventName = keyof Events;

export const emit = <T extends EventName>(
  name: T,
  payload: Events[T]["payload"]
  ) => {
    return e(name, payload);
};

export const on = <T extends EventName>(
  name: T,
  handler: Events[T]["handler"]
  ) => {
    if (handler) return o(name, handler);
};
```
