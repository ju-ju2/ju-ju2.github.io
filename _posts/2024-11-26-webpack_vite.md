---
layout: post
title: Webpack VS Vite
subtitle: 떠오르는 Vite의 이유있는 이유!
categories: Vite
tags: [bundler]
image: /assets/images/20241126_webpack_vite.png
---

### 😵 들어가며

---

왜 우리는 웹팩대신 비트를 선택했을까?
현재 구축하고 있는 라이브러리에서 비트를 처음 사용해보았다. 물론 회사에서도 처음 시도하였다. vite를 채택한데는 이유가 있을 것인데, 오늘은 그 이유에 대해 정리해보고자한다.

<br/>
<br/>

### Webpack VS Vite

---

<em style='font-size: 17px; color: #ff5100; font-weight: bold;'>Webpack</em>

Webpack은 자바스크립트 파일과 그 의존성을 분석하여 하나의 번들 파일로 묶는 방식을 사용한다. 이는 브라우저에 전달할 파일 수를 줄이고 네트워크 요청을 최적화하려는 목적이 있지만, 다음과 같은 단점이 있다:

1. **전체 번들링의 비용**  
   Webpack은 프로젝트의 모든 파일을 분석한 뒤 하나의 번들 파일을 생성한다. 작은 코드 수정이 발생하더라도 전체 파일을 다시 번들링해야 하기 때문에 빌드 속도가 느려질 수 있다.
2. **번들 크기 증가**  
   프로젝트 규모가 커질수록 의존성 파일의 개수와 크기가 증가하며, 초기 로드 속도를 저하시킨다.
3. **Interpreted Language의 한계**  
   Webpack은 자바스크립트로 작성되어 한 줄씩 실행되는 **인터프리터 언어** 특성을 가지며, 대규모 프로젝트에서는 처리 시간이 길어질 수 있다.
4. **설정의 복잡성**  
   다양한 요구사항을 충족하려면 Webpack 설정 파일이 복잡해지며, 학습 곡선이 높다.

<br/>
<br/>

<em style='font-size: 17px; color: #ff5100; font-weight: bold;'>Vite</em>

**Vite**는 Webpack의 단점을 개선할 수 있는 빌드 도구로, 다음과 같은 주요 장점이 있다

1. **ESM(ES Modules) 기반 로드**  
   Vite는 브라우저의 네이티브 **ES 모듈** 지원을 활용하여 필요한 모듈만 개별적으로 로드한다. 즉, `<script type="module">` 태그를 통해 브라우저가 필요한 파일을 바로 요청하므로, 전체 번들링 과정이 불필요하다. 코드 수정 시에도 변경된 파일만 다시 로드하기 때문에 **Hot Module Replacement(HMR)** 속도가 뛰어나다.
2. **초기 빌드 속도 개선**  
   Vite는 전체 번들링을 생략하고 필요한 파일만 처리하므로 개발 서버 시작 속도가 매우 빠르다.
3. **간단한 설정**  
   Webpack에 비해 설정이 간단하고, 기본 제공 기능만으로도 많은 요구사항을 충족할 수 있다.
4. **CRA 대체로 부상**  
   Create React App(CRA)이 공식적으로 더 이상 지원되지 않으면서, 많은 개발자들이 Vite를 선택하고 있다.

<br/>
<br/>

### Vite 후기

---

실제로 지난 프로젝트때는 웹팩 환경이였고 꽤나 파일이 많은 큰 플젝이였는데 실행 속도가 정말 느렸다. 사실 나는 크게 느끼지 못하였는데, 성능이 조금 떨어지는 pc 환경에서 구동되는건 정말 세월아 네월아였다.
그리고 지금 체감 해보는 비트 속도는 정말 굿,,
웹팩에 비해 설정이 덜 까다로운 점도 장점인것같다.
하지만 파일을 계속 브라우저에 요청하는 시스템이라 크기가 커지면 성능이 오히려 내려갈 수 있다는 단점이 있다고 하는데, 프로젝트의 규모에 따라 어떤 빌드 도구를 사용해야할지 선택을 잘 해야할 것 같다. 어쨋던 많은 개발자들이 Vite로 눈을 돌리고 있다고 하는데 나도 이렇게 경험해봐서 다행이다.😀