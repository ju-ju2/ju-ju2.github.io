---
layout: post
title: echart tree shaking
subtitle: 다 털어버렷 🌲🍃
categories: trouble_shooting
tags: [echart, treeshaking]
image: /assets/images/20240325_treeshaking.png
---

#### 들어가며😃

---

프로젝트에서 차트를 보여주는 페이지를 내가 담당하게 되었다. 다양한 라이브러리가 후보군에 있었지만 표현해야했던 그래프 종류를 모두 포함하고 있었고 가장 정보를 많이 얻을 수 있었던 echart 라이브러리를 채택하였다. 해당 라이브러리를 사용하면서 트리쉐이킹 관련 문서를 많이 접할 수 있었는데, 이는 echart 공식 문서에서도 tree shaking API를 사용하라고도 나와있었다. 여튼 트리쉐이킹 관련 정보를 찾아보며 다른 부가적인 정보들도 알게되어 정리하고자 하는 오늘의 포스팅이다.🍃

<br/><br/>

#### 1. Tree Shaking 이란?

---

<span style="color: #ff5100;"> Tree shaking은 JavaScript의 모듈 번들링 도구에서 사용되는 최적화 기법</span> 중 하나로, 번들링할 때 사용되지 않는 코드를 제거하여 번들 크기를 최소화한다.

<p style="color: lightgray; font-size: 12px"> *번들링: 여러개의 파일을 합치는 과정</p>

일반적으로 JavaScript 프로젝트에서는 여러 개의 모듈을 사용하게 되는데, 이 모듈들 중에서 실제 프로젝트에 사용되지 않는 코드들이 존재할 수 있다. 이런 경우 번들링할 때 해당되지 않는 코드까지 모두 포함하여 번들 크기가 커지게 되는 것이다.

그래서 Tree shaking 과정을 통해 모듈 간의 의존 관계를 분석하여 사용되지 않는 코드를 식별하고 제거하여 **최종 번들에는 프로젝트에 실제로 사용되는 코드만 포함되로독 함으로써 프로젝트의 성능을 높일 수 있는 것**이다.
<br/><br/>

#### 2. Echart Tree Shaking API

---

가장 기본적으로 Echart 를 사용하는 방법은 아래와 같다.

```typescript
import React from "react";
import type { EChartsOption } from "echarts";
import ReactECharts from "echarts-for-react";

const EchartPage = () => {
  const chartRef = React.useRef(null);

  const options: EChartsOption = {
    xAxis: {
      type: "category",
      data: ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"],
    },
    yAxis: {
      type: "value",
    },
    series: [
      {
        data: [150, 230, 224, 218, 135, 147, 260],
        type: "line",
      },
    ],
  };

  return (
    <ReactECharts
      ref={chartRef}
      option={options}
      opts={{ renderer: "svg" }}
      style={{ height: 500 }}
    />
  );
};

export default EchartPage;
```

위의 구조는 `echarts-for-react` 에서 제공해주는 모듈들을 모조리 불러오고 있다. 이제 여기서 내가 프로젝트에 쓰는 그래프와 속성들만 불러오도록 변경해보겠다.  
<br/>

```typescript
import {
  BarChart,
  LineChart,
  ParallelChart,
  ScatterChart,
} from "echarts/charts";
import {
  GridComponent,
  LegendComponent,
  LegendPlainComponent,
  MarkPointComponent,
  TooltipComponent,
} from "echarts/components";
import * as echarts from "echarts/core";
import { CanvasRenderer } from "echarts/renderers";
import type { EChartsOption } from "echarts";
import EChartsReactCore from "echarts-for-react/lib/core";
import "echarts/lib/component/markPoint";

echarts.use([
  LineChart,
  CanvasRenderer,
  GridComponent,
  MarkPointComponent,
  TooltipComponent,
  LegendComponent,
  ParallelChart,
  BarChart,
  ScatterChart,
  LegendPlainComponent,
]);

const EchartTreeShaking = () => {
  const options: EChartsOption = {
    xAxis: {
      type: "category",
      data: ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"],
    },
    yAxis: {
      type: "value",
    },
    series: [
      {
        data: [150, 230, 224, 218, 135, 147, 260],
        type: "line",
      },
    ],
  };
  return (
    <EChartsReactCore
      echarts={echarts}
      option={options}
      theme="space-theme"
      style={{ height: 500 }}
      notMerge={true}
    />
  );
};

export default EchartTreeShaking;
```

로직은 꽤 간단하다. 필요한 모듈을 불러내서 echarts.use에 설정만 해주면 끝!

위는 내가 프로젝트에서 실제로 적용했던 코드이다. 불러오는게 많아보이긴 하나 결과적으로는 이전 코드보다 훨씬 크기가 작다는 것...!🫢

[Echart](https://echarts.apache.org/handbook/en/basics/import/) 공식 문서에 따르면 5버전부터 새로운 tree shaking API 를 도입하였다고 한다. 옵션에 따라 어떤 모듈을 가져와야 하는지 더 쉽게 알 수 있도록 import를 할 수 있다하는데 실제로 차트는 차트대로, 속성은 속성대로 불러오는 구조라 코드를 커스텀하기 용이했다.

<em style='font-size: 20px; color: #BAD1C2; font-weight: bold;'>아니, 그래서 크기가 줄었는지 어떻게 확인할 수 있지?🤷🏻‍♀️</em>

<br/><br/>

#### 3. 번들 사이즈 비교\_cra-bundle-analyzer

---

cra-bundle-analyzer는 CRA 프로젝트에서 번들 크기를 분석하고 시각화하는 도구이다. 이를 통해 번들에 포함된 파일들의 크기를 그래프나 트리 맵과 같은 시각적인 형태로 확인할 수 있다.

- 설치 : `npm install --save-dev cra-bundle-analyzer`
- 사용 : `npx cra-bundle-analyzer`

명령어를 수행하면 html 파일이 하나 만들어지는데 아래의 이미와 같은 파일이 생성된다. <p style="color: lightgray; font-size: 12px">오메,,,이와중에 놀라운 존재감을 표하는 faker...</p>
<img width="600" alt="image" src="https://github.com/ju-ju2/ju-ju2.github.io/assets/71650663/72e463a4-3fc9-4b15-a243-ecbb141e4610">

자, 그럼 Echart 의 트리 쉐이킹 전 후 크기를 비교해보자. (Parse size)

<div style="display: flex; gap: 12px">
<img width="250" alt="echart 트리쉐이킹 후" src="https://github.com/ju-ju2/ju-ju2.github.io/assets/71650663/fb377fc1-4fe2-4915-89c7-c89174e1fc9d">
<img width="250" alt="echart 트리쉐이킹 전" src="https://github.com/ju-ju2/ju-ju2.github.io/assets/71650663/4be83501-0fbd-4fec-bc0b-0481e7d4de58"></div>  
<br/>
오호라 트리쉐이킹 성공! 430KB 정도 줄어들었다.😎

<br/><br/><br/><br/>

---

웹 애플리케이션에서는 사용자 경험 측면에서 초기 로딩 속도가 매우 중요하다. 이를 위해 사용되지 않는 코드를 줄여나가는 것이 매우 중요한데 트리쉐이킹이 하나의 방법이 되는 것을 이번 Echart 라이브러리를 커스텀해보면서 알게되었다. 나 이 파트 담당한거 참 잘한 걸지도..? 오늘도 하나 배워간다. 근데 그러고 보니 내 프로젝트에 지금 echart만 트리쉐이킹이 필요한게 아닌듯하다(?) 어쨋던 지식의 영역을 더 뻗치고 있는 이 과정에서 코드 작성 요령 뿐만이 아닌 프로젝트 성능까지 잘.알.이 되기 위해 더 많은 공부를 해야함을 깨달았다. 내친김에 리액트 성능 최적화에 대해 더 알아봐야겠다 후후. 오늘도 부릉부릉하는 나의 궁둥이!! 화이팅~~!🤓
