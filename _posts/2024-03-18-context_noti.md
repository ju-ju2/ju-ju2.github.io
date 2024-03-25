---
layout: post
title: createContext 로 전역 context 설정하기
subtitle: 더이상 사라지지마라 noti
categories: trouble_shooting
tags: [context, antd, notification]
image: /assets/images/20240318_globalContext.png
---

#### 들어가며😃

---

Antd에서 notification이라는 기능을 사용하면서 겪은 이슈를 바탕으로 globalContext에 대한 개념을 익히게 되었다. 조금 더 잘 익혀두기 위해 정리해보는 오늘의 포스팅이다.

<br/><br/>

#### 1. 이슈 발생: noti가 사라진다?!

---

[Antd](https://ant.design/components/notification) 문서에서 제공해주는 샘플대로 페이지를 만들고 노티를 생성해보았다.

```typescript
const NotificationDetail = () => {
  const [api, contextHolder] = notification.useNotification();

  type NotificationType = "success" | "info" | "warning" | "error";

  const onClickOpenNotification = (type: NotificationType = "success") => {
    api[type]({
      message: "Notification",
      description: "This is the content of the notification. ",
    });
  };

  return (
    <>
      {contextHolder}
      <Space size={"small"}>
        <Typography.Text>상세페이지</Typography.Text>
        <Button onClick={() => onClickOpenNotification()}>notification</Button>
      </Space>
    </>
  );
};

export default NotificationDetail;
```

<img width="500" alt="noti" src="https://github.com/ju-ju2/ju-ju2.github.io/assets/71650663/047a8677-2a8c-4ae9-bcbb-dbe407bb3516">  
<br/>
결과는 아주 나이스! 이쁜 노티가 아주 잘뜬다.
그런데 나는 노티를 띄우고 페이지 이동이 필요하여 로직을 수정해보았다.

<br/><br/>

```typescript
const NotificationDetail = () => {
  const navigate = useNavigate();
  //...
  const onClickOpenNotification = (type: NotificationType = "success") => {
    //...
    navigate(-1);
  };
};
```

<img width="500" alt="noti fadeout" src="https://github.com/ju-ju2/ju-ju2.github.io/assets/71650663/02db8d9a-7a98-41cb-97be-5665b54176bb"> 
<br/>
읭? 스럽게도 사라지기는 커녕 보이지도 않는 노티다..
<br/>
위의 코드를 살펴보면 <u>`contextHolder` 라는 context 아래 노티들이 관리</u>되어져보인다. 그렇다면 나는 노티가 페이지 이동에 상관없이 모든 페이지에서 보일 수 있도록 **페이지 상위에 기능을 정의**해 줄 필요가 있음을 깨달았다.

<br/><br/>

#### 2. createContext

---

우선 src 폴더 아래 context 파일을 만들었고 해당 파일안에 컨텍스트를 생성하였다.

```typescript
import { notification as AntdNotification } from "antd";
import { NotificationInstance } from "antd/es/notification/interface";
import React, { createContext } from "react";

interface NotificationType {
  // notification의 내용 부분
  message: string;
  description?: string;
  key?: React.Key;
}

interface GlobalContextContainerProps {
  children?: JSX.Element;
}

interface GlobalConfig {
  notification: ({}: NotificationType) => void;
  notificationApi: NotificationInstance;
}

export const GlobalContext = createContext<GlobalConfig | undefined>(undefined);

const GlobalContextContainer = ({
  children,
}: React.PropsWithChildren<GlobalContextContainerProps>): React.ReactElement => {
  const [notificationApi, contextHolder] = AntdNotification.useNotification();

  const notification = ({ message, description, key }: NotificationType) => {
    notificationApi.open({
      message,
      description,
      key,
    });
  };

  const value = {
    notification,
    notificationApi,
  };

  return (
    <GlobalContext.Provider value={value}>
      {contextHolder}
      <div>{children}</div>
    </GlobalContext.Provider>
  );
};
```

`Context`에 대한 구조를 우선 보면 위 코드에서 **<GlobalContext.Provider value={value}>** 부분은 React의 Context API를 사용하여 전역 상태를 관리하는 역할을 한다.  
여기서 value 객체는 `notification` 함수와 `notificationApi` 객체를 포함하고 있다. value 객체가 GlobalContext의 값으로 설정됨으로써 이
<span style="color: #ff5100;"> GlobalContext를 구독하는 컴포넌트(`children`에 속한 페이지)들은 notification 함수와 notificationApi 객체에 접근</span>할 수 있게되는 것 이다.

<br/><br/>

#### 3. useContext

---

위의 코드에 아래의 코드를 추가하였다.

```typescript
// ...GlobalContextContainer
// ...
export const useGlobalContext = () => {
  const context = React.useContext(GlobalContext);
  if (context === undefined) {
    throw new Error(
      "GlobalContext must be used within a GlobalContextProvider"
    );
  }
  return context;
};
```

`React.useContext(GlobalContext)`: React의 useContext 훅을 사용하여 GlobalContext에서 제공하는 전역 상태(value)를 가져오는 로직이다. 이렇게 함으로써 GlobalContext가 제공하는 값(notification, notificationApi)에 접근할 수 있다.

<br/><br/>

#### 4. Context 감싸기

---

위에서 GlobalContext를 구독하는 컴포넌트들은 컨텍스트가 제공하는 값들을 공유할 수 있다했다. 그렇게 하기 위해 우리가 사용하는 페이지 상위에 컨텍스트를 정의해주어야 한다.

```typescript
import { BrowserRouter } from "react-router-dom";
import Router from "./routes";
import "./scss/main.scss";
import GlobalContext from "context/GlobalContext";

function App() {
  return (
    <BrowserRouter>
      <GlobalContext>
        <Router />
      </GlobalContext>
    </BrowserRouter>
  );
}

export default App;
```

위의 코드에서 `Router`부분이 우리가 이동하는 페이지며 그 상위에 GlobalContext를 감싸줌으로써 어떤페이지에서든지 notificationApi를 쓸 수 있게된다.

<br/><br/>

#### 5. 실사용

---

나는 useGlobalContext 훅을 noti를 발생시킬 페이지에서 사용해보았다.

```typescript
import { Button, Space, Typography } from "antd";
import { useGlobalContext } from "context/GlobalContext";
import { useNavigate } from "react-router-dom";

const NotificationDetail = () => {
  const navigate = useNavigate();

  const { notification } = useGlobalContext();
  const onClickOpenNotification = () => {
    notification({ message: "노티!😵" });
    navigate(-1);
  };
  return (
    <Space size={"small"}>
      <Typography.Text>상세페이지</Typography.Text>
      <Button onClick={() => onClickOpenNotification()}>notification</Button>
    </Space>
  );
};

export default NotificationDetail;
```

<br/>
그리고 최종화면 짠!

<img width="500" alt="noti" src="https://github.com/ju-ju2/ju-ju2.github.io/assets/71650663/5ca7ff79-8e74-4d63-94c3-bb22764917ef">

<br/>
이전에는 페이지 이동이 발생하면 노티가 보이지도 않았는데 전역 context를 적용하니 어느 페이지에서도 노티가 잘 보이는 것을 확인했다! 으하하

<br/><br/><br/><br/>

---

처음에 노티가 사라지는 현상에 대한 원인은 어느정도 쉽게 파악할 수 있었다. 그리고 페이지 상단에서 관리해줘야 되는구나!라도고 쉽게 떠올렸다. 하지만 context에 익숙하지 않은 터라 실제 코드로 어떻게 구현해야하는지 막막했었고 그 과정도 쉽지만은 않았다. 그래도 이렇게 정리하고 보니 조금 더 이해가 잘 되는것같다. 앞으로도 비슷한 이슈가 발생하면 더 잘 대처할 수 있기를! 빠샹~🤷🏻‍♀️👊🏻
