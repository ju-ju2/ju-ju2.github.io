---
layout: post
title: 깃허브 블로그에 댓글창 만들기 with utterances
subtitle: utterances 쓸까, disqus쓸까 고민이라면 utterances 쓰세요
categories: github
tags: [blog, github, disqus, utterances]
og:image: "/assets/images/banners/juju-og-image.png"
---

<p style="font-size = 16px">
깃허브 블로그에 댓글 창 만드는 법을 포스팅 하려고 한다.<br />
사실 유튜브 보다가 disqus로 먼저 댓글창을 만들었지만,,, 다른 글들을 찾아보다 disqus는 광고가 심하고 무겁다는 등의 평으로 github 랑 연동되는 utterances 라는 플랫폼으로 많이 갈아타고 있는것을 알았다.<br />
그렇다면 나도 해야지!! github로 만든 블로그에 github로 알림오는 댓글,, 너무 깐지나쟈나..!?🤩
</p>

😎그렇다면 바로 시작해보자!

### _1. utterances 설치하기_

[utterances](https://github.com/apps/utterances) 페이지에 들어가서 install 버튼을 눌러준다.

![install image](/assets/images/../../../assets/images/2023-04-07-make-comment/utterances/1.png "install")

### _2. repositories 선택하기_

install 을 누르면 어디 저장소에 설치할 것이냐고 묻는다. 나는 깃허브 블로그에만 적용할 것이니 only select repositories 에서 해당 레파지토리를 선택했다. 그리고 바로 install 버튼 클릭!

![repositories image](/assets/images/../../../assets/images/2023-04-07-make-comment/utterances/2.png "repositories")

### _3. configuration 설정하기_

그다음으로는 댓글이 깃허브 이슈에 올라온다는 글과 어떻게 작동하는지 알려주는데 조금 읽어보다가 configuration 가서 마저 진행한다.
<br />

**3-1. repo**
<br />
repo란에는 깃허브 username과 연동하는 repository의 이름을 적어주면 된다.

![configuration repo](/assets/images/../../../assets/images/2023-04-07-make-comment/utterances/3.png "configuration repo")

**3-2. Issue Mapping**
<br />
댓글이 달리면 이슈에 어떻게 보여줄건지 선택한다.
나는 pathname으로 내가 작성한 폴더 명으로 지정했다. 제목에 날짜가 있어서 보기 편할 것 같았다.

**3-3. Issue Label**
<br />
이건 옵션사항이라 따로 선택하지 않았다.

**3-4. Theme**
<br />
역시 기본으로 설정했다. 기본이 짱이다.

![configuration Issue](/assets/images/../../../assets/images/2023-04-07-make-comment/utterances/4.png "configuration Issue")

### _4. script 설정하기_

위의 설정을 마쳤으면 스크립트가 짠!하고 나온다. 이걸 복사해서 파일에 설정해주면 된다.
나같은 경우엔 /\_layouts/post.html에 복붙해줬다

![script](/assets/images/../../../assets/images/2023-04-07-make-comment/utterances/5.png "script")

### _5. 결과_

#### _5-1. 1차 결과_

자 이제 저장하고 깃허브에 푸시하면!!!!!! 짜자잔!😆

![test](/assets/images/../../../assets/images/2023-04-07-make-comment/utterances/6.png "tset")

#### _5-2. 레이아웃 수정(옵션)_

....어라? 근데 뭐가 좀 이상하다. 비율이 아주 맘에 안든다..이런거 용납 못한다.<br />
찾아보니 css로 설정할 수 있었다.

```css
.utterances {
  max-width: 100% !important;
}
```

요거를 적용해주면 되는데 나같은 경우엔 /\_sass/yat.scss 에 추가했다. 사용자마다 사용하는 테마가 달라 다른 위치에 저장될 수 도 있다.

이제 다시 푸시해보면!!!!!
<br />
<br />
<br />
따흥😆>
<br />
<br />
<br />

#### _5-3. 최종결과_

![success](/assets/images/../../../assets/images/2023-04-07-make-comment/utterances/7.png "success")

뭐가 변한거야? 싶기도 하겠지만 진짜진짜 변했다. 너비가 아주 마음을 편하게 해준다.ㅎ 사이다다.

### _6. 테스트해보기_

댓글을 적어보면 깃허브 레파지토리의 이슈에서 확인할 수 있다.<br />
아까 설정했던 파일경로로 아주 잘 들어왔다. 뿌듯🥹

![comment](/assets/images/../../../assets/images/2023-04-07-make-comment/utterances/9.png "comment")

<br />

이제 글만 잘 적으면 된다. 열심히 기록 잘하자🔥
