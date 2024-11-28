---
layout: post
title: github action으로 build 자동화하기
subtitle: 깃허브 액션 라인 파보기
categories: github
tags: [github]
image: /assets/images/20241128_github_action.png
---

### 😵 들어가며

---

Github Actions은 Github 레포에서 개발 워크플로우를 자동화할 수 있는 도구이다. 이건 여담이긴 하지만 사실,, 최근 모노레포를 사용하다 특정 패키지에서 다른 패키지의 컴포넌트를 인식하지 못해 원인을 찾느라 엄청 헤맨 경험이 있었다..근데 알고보니 누군 또 되서 이상하게 여겼는데 알고보니 내가 빌드하지 않고 컴포넌트를 참조하려 해서 그랬던 것이였다.. 매우 바보같은 사람🤪ㅠ

여튼 한차례 챙피함을 느끼고 우리 패키지 구조상 빌드에 대한 중요성을 배웠다. 사수님이 프로젝트 초기에 빌드 체크하고 커밋하라고도 하셨는데 아차차~ 역시 울 사수님 틀린만 하나없다~

그래서 언급해보는 **자동화 로직의 중요성!**
이런 상황해서 따로 build 하지않아도 push하면 알아서 build되면 얼마나 좋게요?!
그겸, 라이브러리를 cdn으로 배포하면서 깃허브 액션을 처음 설정해보았고 그 내용을 간단하게 정리해보겠다는 오늘의 포스팅!✍🏻

<br/>
<br/>

---

우선 파일을 생성해준다. .github/workflows/{name}.yml 형식을 따라야 깃허브 액션이 인식할 것 이다!
<br/>

<img width="300" alt="dist생성" src="https://github.com/user-attachments/assets/a4b9cfa1-82e5-4877-bc68-b747c3442b5a">

<br/>

그리고 아래의 내용을 첨부하였다.

```
name: Build and Deploy



on:

push:

branches:

- main # main 브랜치에 푸시될 때 빌드 트리거

pull_request: # PR 생성 시에도 빌드 확인

branches:

- main



jobs:

build:

name: Build Project

runs-on: ubuntu-latest



steps:

# 1. 저장소 체크아웃

- name: Checkout Repository

uses: actions/checkout@v4



# 2. Node.js 설정

- name: Setup Node.js

uses: actions/setup-node@v4

with:

node-version: "20" # 프로젝트 Node.js 버전



# 3. 종속성 설치

- name: Install Dependencies

run: yarn install



# 4. 빌드 실행

- name: Build Project

run: yarn build



# 5. dist 디렉토리 커밋 및 푸시

- name: Commit dist folder

run: |

git config --global user.name "GitHub Actions"

git config --global user.email "actions@github.com"

git add dist/

git commit -m "Add dist files"

git push origin main

env:

GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

```

<br/>

내가 구현한 플로우는 주석으로 거의 이해할 수 있다. 나의 목적은 푸시 후 빌드가 되는 것이였기 때문에 간단하다!
저장소 체크아웃, 노드 설정은 기본으로 하는것 같고, 우선 install을 해주어야한다. 깃 레포에는 node_modules가 포함되지 않기에 설치 후 빌드를 해준다. 그리고 커밋과 푸시를 하면 되는데 위의 내용을 따로 변경하지 않아도 깃허브에서 알아서 email, name을 할당해주는 듯 하다.

<br/>

하지만 이대로 하면 에러가 날 것이다! 아마 워크플로우 권한이 read로만 되있을건데 아래와 같이 수정해서 다시 푸시하면 된다!

<br/>

**깃허브 레포 -> setting -> actions -> general -> workflow permissions**
<br/>

<img width="500" alt="permissions" src="https://github.com/user-attachments/assets/118d47a7-1a8b-4409-afad-e35742c4fb98">

<br/>

그리고 잘 진행된다면 actions 탭에 진행 경과가 보일 것이고, 성공하면 이쁘게 dist 파일이 생길 것이다.

<br/>

<img width="500" alt="workflow" src="https://github.com/user-attachments/assets/5e122fa8-5cfc-4b5a-bb2f-5e1ee2160ed0">

중중중 짠~

<br/>

<img width="500" alt="dist생성" src="https://github.com/user-attachments/assets/2acadd25-1a68-4ceb-85b1-bfaaebfa2f8f">

<br/>

간단하게 build만 자동화하는 로직을 구현해보았는데, 생각보다 어렵지 않았던 것 같다. 지금은 빌드까지만이였지만 주로 배포까지 다음엔 더 진행해보아야즹😇
