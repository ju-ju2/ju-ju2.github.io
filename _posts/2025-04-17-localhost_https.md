---
layout: post
title: localhost에서 https 사용하기
subtitle: feat.mkcert
categories: Vite
tags: [Vite]
image: /assets/images/20250417_mkcert.png
---

### 😵 들어가며

---

외주사에서 작업한 프로젝트 중 관리자 프로그램의 스크립트가 다른 프로젝트와 다르길래 확인해보았더니 localhost 에서 https 사용을 위한 설정임을 알았다.

로컬 개발 환경에서 https 를 사용하는 이유는 여러가지일 수 있다. 배포 환경과 동일하게 구성하여 올바른 테스트를 위해서 일 수 있고, 소셜로그인 혹은 외부 API 사용 시 https 가 아니면 거절하는 경우 등..
이러한 상황에서 로컬환경을 쉽게 https 로 구현할 수 있는 도구가 mkcert인데 오늘은 해당 도구가 무엇인지, 어떻게 설정하는지, 사용 방법에 대해 포스팅 해보려 한다.

<br/>
<br/>

### 🔐 본문

---

1. mkcert 이란?

   로컬에서 HTTPS 개발환경을 쉽게 만들 수 있게 도와주는 **로컬 SSL 인증서 생성 도구** 이다.
   `https://localhost`에서 복잡한 OpenSSL 설정 없이 바로 사용 가능하며 CORS, 서비스워커, OAuth https 환경에서 발생할 수 있는 문제를 미리 체크 가능하다.

   <br/>

2. mkcert 설치 (macOS 기준)

   `brew install mkcert`
   터미널 열고 Homebrew로 mkcert를 설치한다.

   <br/>

3. 로컬 루트 인증기관(CA) 생성

   `mkcert -install`
   해당 명령어로 브라우저가 요청을 신뢰할 수 있도록 만들어준다. 이 설정은 초기에 한번만 하면 된다!

   <br/>

4. 프로젝트에 인증서 생성

   `mkcert localhost`
   해당 명령어를 https 설정을 원하는 프로젝트에서 실행시키면 두가지 파일이 생성된다.
   localhost.pem(인증서), localhost-key.pem(비공개 키)
   이 파일을 바탕으로 개발 서버를 실행시키면 끝이다!

   <br/>

5. 개발 환경 실행
   "scripts": { "start": "cross-env HTTPS=true SSL_CRT_FILE=./localhost.pem SSL_KEY_FILE=./localhost-key.pem react-scripts start", }

   <br/>
   <br/>
