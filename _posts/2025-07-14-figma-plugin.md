---
layout: post
title: Figma 플러그인 React + Vite + ts 초기 세팅
subtitle: 자동화를 위한 멀고도 험한 길
categories: Figma
tags: [Vite, Figma]
image: /assets/images/20250710_figma_plugin.png
---

### 😵 들어가며

---

이전 회사에서 피그마에 정의된 토큰을 프로젝트 코드에서 사용할 수 있도록 원하는 형태로 가공해서 푸시하는 피그마 플러그인을 개발한 적이 있었다. 사실 A to Z 까지 내가 한게 아니라 기존에 하던 프로젝트를 받아서 진행한거라 모든 로직을 이해했다고 하기엔 어려움이 있다... 그런데 사이드 프로젝트나 새로운 회사에서 작업을 할때마다 비슷한 기능을 가진 플러그인이 필요할때가 있고, 또 입맛에 맞게 가공하려면 내가 처음부터 끝까지 개발할 줄 알아야 한다는 생각이 들었다. 그래서 초기 세팅을 시작하는 중이고 그 과정을 기록해보려한다.

<br/>

++ 일단 [공식문서](https://www.figma.com/plugin-docs/) 공식문서 참고하시라,,,

<br/>
<br/>

### 플러그인 초기 템플릿 생성

---

Figma 앱에서 Plugins > Development > New Plugin을 선택하면 플러그인의 초기 구조와 manifest.json이 자동으로 생성된다. 이때 manifest.json을 살펴보면 다음과 같이 플러그인의 실행 구조를 파악할 수 있다:

`main`: Figma API를 사용하는 백엔드 역할의 코드 (code.js)

`ui`: 사용자 인터페이스를 담당하는 HTML 파일 (ui.html)

<br/>
<br/>

### React + Vite 기반 개발 환경 구성

---

나는 리액트 프로젝트로 플러그인을 만드려고 했는데 기본 제공 템플릿에 vite를 얹기보다는, Vite 템플릿을 새로 생성하여 그 위에 Figma 플러그인 설정을 얹는 방식이 더 편리하다고 판단했다.

```
yarn create vite
# 템플릿 선택: react → TypeScript
```

Figma 플러그인 타입을 지원하기 위해 다음 패키지를 설치했다:

`yarn add -D @figma/eslint-plugin-figma-plugins @figma/plugin-typings`

그리고 tsconfig.json에 Figma 타입을 인식시키기 위해 typeRoots를 설정했다

`"typeRoots": ["./node_modules/@types", "./node_modules/@figma"]`

<br/>
<br/>

### 프로젝트 디렉토리 구조

---

```
src
 ┣ code
 ┃ ┗ code.ts         ← Figma API를 다루는 main thread 코드
 ┗ ui
 ┃ ┣ assets          ← UI에 필요한 리소스
 ┃ ┣ App.tsx         ← React UI 루트 컴포넌트
 ┃ ┣ index.html      ← iframe으로 렌더링될 HTML
 ┃ ┣ main.tsx        ← React 진입점
 ┃ ┗ vite-env.d.ts   ← Vite 환경 타입 설정

기타 루트 파일:
 ┣ esbuild.code.cjs  ← code.ts를 위한 별도 빌드 설정
 ┣ vite.config.ts    ← Vite UI 빌드 설정
 ┣ manifest.json     ← Figma 플러그인 메타 정보
 ┣ tsconfig.json 외 tsconfig.app.json, tsconfig.node.json

```

<br/>
<br/>

### Vite 설정 (vite.config.ts)

---

```
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import { viteSingleFile } from "vite-plugin-singlefile";

export default defineConfig({
  root: "./src/ui", // UI 소스 위치
  plugins: [react(), viteSingleFile()],
  build: {
    target: "esnext",
    minify: "esbuild",
    outDir: "dist",
    rollupOptions: {
      output: {
        inlineDynamicImports: true, // 코드 분할 방지
      },
    },
    commonjsOptions: {
      transformMixedEsModules: true,
    },
  },
});

```

주요 설정 설명
viteSingleFile: Vite는 기본적으로 JS, CSS, 이미지 파일 등을 분리해 출력하지만, Figma 플러그인의 UI는 외부 파일 로딩을 허용하지 않기 때문에 단일 HTML 파일로 모든 내용을 인라인하는 플러그인이 필요하다.

inlineDynamicImports: 코드가 여러 청크로 분할되지 않도록 설정. 이 설정이 없으면 ui.html 내부에서 스크립트 로딩 오류가 발생할 수 있다.

<br/>
<br/>

### esbuild 설정 (esbuild.code.cjs)

---

Vite는 UI 빌드에만 사용되기 때문에, Figma의 main thread에서 동작하는 code.ts는 별도의 빌드 파이프라인을 구성해야 한다. 이를 위해 esbuild를 사용하여 빠르고 간단하게 code.ts를 번들링했다.

```
const esbuild = require("esbuild");

esbuild
  .build({
    entryPoints: ["src/code/code.ts"],
    bundle: true,
    outfile: "dist/code.js",
    minify: true,
    target: "es6",
  })
  .then(() => {
    console.log("✅ code.ts 빌드 완료");
  })
  .catch(() => process.exit(1));

```

<br/>
<br/>

### manifest.json 설정

---

최종적으로 빌드된 결과물(dist/code.js, dist/index.html)을 Figma 플러그인에서 인식시키기 위해 manifest.json을 다음과 같이 작성했다.

```
{
  "name": "variables-to-code",
  "id": "variables-to-code",
  "api": "1.0.0",
  "main": "dist/code.js",
  "ui": "dist/index.html",
  "editorType": ["figma"]
}

```

<br/>
<br/>

### 빌드 스크립트 설정 (package.json)

---

```
"scripts": {
  "build": "tsc -b && yarn build:ui && yarn build:code",
  "build:ui": "vite build --emptyOutDir=false",
  "build:code": "node ./esbuild.code.cjs"
}

```

`tsc -b`: TypeScript 프로젝트를 빌드 (멀티 tsconfig 구조에서 필수)

`build:ui`: Vite로 UI를 빌드. --emptyOutDir=false 옵션을 통해 code.js가 덮어써지는 것을 방지

`build:code`: esbuild를 통해 code.ts를 code.js로 번들링

<br/>
<br/>

### 마무리

---

호오,,,일단 했다,,,초기 설정 완료!👍
