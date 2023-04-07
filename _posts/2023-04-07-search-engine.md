---
layout: post
title: 깃허브 블로그 검색엔진 등록 (구글, 네이버)
subtitle: 많이 들어오시라.
categories: github
tags: [blog, github]
---

깃허브 블로그 만드는 게 보통 일이 아니였다,,삽질의 연속 그리고 그것은 현재진행형!!!
<br />
그런데 이렇게 힘들게 만든 블로그가 검색이 안된다면 매우 슬플것이다🥲
<br />
그래서 내 사이트를 검색엔진에 등록해보려한다. 구글, 네이버, 다음 등등이 있겠지만 나는 구글, 네이버만 등록했다.
<br />
절차는 따라오시라!!
<br />
<br />

### _1. 구글 등록하기_

[구글 서치 콘솔](https://search.google.com/search-console/about)에 들어가 시작하기를 누르면 아래와 같은 화면이 나온다.
<br />

![google1](/assets/images/2023-04-07-search-engine/2.png "google1")
<br />
두번째 URL접두어 부분에 깃허브 블로그 주소를 입력한다.
<br />
<br />

다음은 소유권 확인을 위한 단계이다.
<br />
파일을 다운받고 **확인버튼을 누르기전에!!**
<br />

![google2](/assets/images/2023-04-07-search-engine/4.png "google2")
<br />

파일을 깃허브 블로그 파일에 저장해준다. 위치는 \_config.yml이 있는 가장 바깥위치에 놓아야한다. **위치 중요!!**
<br />

![google3](/assets/images/2023-04-07-search-engine/3.png "google3")
<br />
그리고 저장소에 푸시해주고 반영이 됐을까 싶을때 확인 버튼을 눌러준다.
<br />
<br />

그러면 이렇게 소유권이 확인되었다는 확인 버튼이 뜬다.
<br />

![google5](/assets/images/2023-04-07-search-engine/5.png "google5")
<br />

Search Console 시작하기
<br />

![google6](/assets/images/2023-04-07-search-engine/6.png "google6")
<br />
<br />
<br />

왼쪽 색인생성 카테고리에서 Sitemaps를 선택한다.
<br />

새 사이트 맵 추가 란에 **깃허브 주소/sitemap.xml** 을 제출하고 조금만 기다리면 성공이다!
<br />

![google7](/assets/images/2023-04-07-search-engine/7.png "google7")
<br />
<br />
<br />

주소창에 **깃허브 주소/sitemap.xml** 을 쳐보고 아래와 같은 화면이 나오면 끝이다. 왕간단 😎
<br />

![google8](/assets/images/2023-04-07-search-engine/8.png "google8")
<br />
<br />
<br />

### _2. 네이버 등록하기_

네이버도 똑같다.
<br />

[네이버 서치 어드바이저](https://search.google.com/search-console/about)에 들어가 로그인 후 우측 상단의 웹마스터 도구 버튼을 클릭한다.
<br />

![naver1](/assets/images/2023-04-07-search-engine/9.png "naver1")
<br />
<br />
<br />
<br />
구글이랑 똑같이 파일을 다운받을 수도 있고, 메타태그를 추가하는 두가지 방식이 있다.
<br />
나는 구글이랑 똑같은 방식으로 진행했다.

![naver2](/assets/images/2023-04-07-search-engine/10.png "naver2")
<br />
<br />

![naver2](/assets/images/2023-04-07-search-engine/14.png "naver2")
<br />
똑같이 \_config.yml이 있는 위치에 놔두고 저장소에 push!!!
<br />
<br />
<br />

![naver2](/assets/images/2023-04-07-search-engine/11.png "naver2")
<br />
나에게 주어진 합격목걸이😎  
<br />
<br />

다시 사이트 목록에 있는 내 깃허브 블로그를 선택해준다.

<br />

![naver3](/assets/images/2023-04-07-search-engine/16.png "naver3")
<br />

<br />

요청 → 사이트맵 제출로 가서 **깃허브 주소/sitemap.xml** 를 입력!

![naver3](/assets/images/2023-04-07-search-engine/12.png "naver3")

 <br />

검증 → robots.txt 에서 수집요청 버튼을 누르면 되는데 내용이 채워질 때까지 기분탓인지 좀 오래걸렸다.

![naver3](/assets/images/2023-04-07-search-engine/12.png "naver3")

 <br />

수집요청에 성공했다면 끝이다. 역시 왕간단.
<br />
<br />
<br />
끝~
