---
layout: post
title: title: Github shh 사용하기
subtitle: 어디서나 보증된 나!
categories: Github
tags: [Github]
image: /assets/images/20250422_ssh.png
---

### 😵 들어가며

---

회사에서 개인 계정과 회사 계정을 번갈아가면서 써야하는 경우가 있었다. 그런데 vscode 계정을 하나로 쓰다보니 다른 계정의 clone/pull/push 에 문제가 생겼고 이를 해결하기 위해 SSH 방식을 택했다. 사실 이게 뭔지 제대로 공부하지 않고 쓴 것 같아서 해당 인증의 원리와 방식에 대해 공부하고 정리할 필요가 있어보여 정리하는 오늘의 포스팅이다.

<br/>
<br/>

### 🔐 본문

---

1.  SSH 란?

    SSH란 Secure Shell의 약어로 컴퓨터끼리 보안 연결을 통해 통신할 수 있도록 해주는 프로토콜이다.
    😃 쉽게 말하면 내 컴퓨터를 인증된 사용자로 등록하여 비밀번호 없이 안전하게 로그인해서 명령을 주고받을 수 있게 해주는 방식인데, 깃허브에서는 이를 통해 별다른 인증없이 안전하게 clone/pull/push 를 할 수 있다.

    <br/>

2.  HTTPS, SSH 클론의 차이점

    깃허브에서는 저장소를 클론할 때 두가지 방식이 있다. https, ssh 방식! 둘의 차이는 인증에 있는데 https 방식은 매번 계정을 인증할 필요가 있는데 ssh 방식은 한번 인증하면 계속 사용가능하다는 점이다. 또한 https 는 url를 그대로 복사해서 쓰기 편하고 ssh는 따로 설정해준 이름으로 클론해주어야한다. 해당 방식에서는 아래에서 따로 더 풀어보겠다.

    <br/>

3.  SSH Key 만들기

    `ssh-keygen`
    해당 명령어를 입력하면 id_rsa(프라이빗 키), id_rsa.pub(공개키) 두개의 키가 쌍으로 만들어진다.

    `cd ~/.ssh`
    생성된 두개의 키를 확인한다.

    `cat id_rsa.pub`
    공개키의 문자열을 복사해서 깃허브 계정에 등록해준다.
    settings -> SSH and GPG keys -> new SSH key

    `nano ~/.ssh/config`
    나는 하나의 컴퓨터에 여러개의 계정을 둘 것이기에 추가 설정이 필요했다. 각 키에 이름을 붙여주는 작업을 아래와 같이 했다.

    Host github.com-personal  
    HostName github.com  
    User git
    IdentityFile ~/.ssh/id_ed25519_personal

    <br/>

    Host github.com-work  
    HostName github.com  
    User git  
    IdentityFile ~/.ssh/id_ed25519_work

    <br/>

    `ssh -T (HostName)`

    ssh -T 뒤에 알맞은 호스트 네임을 붙여주고 hello~ 가 나온다면 인증 성공!

    <br/>
    <br/>

---

위와 같이 설정만 해주면 계정과 상관없이 저장소 접근이 가능해진다! 아주 안전하게 아주 간편하게!
입사하고 이게 뭐지 하면서 설정했었는데 정리를 하니 더 알아간 느낌이다. 역시 정리의 힘이란!

   <br/>
