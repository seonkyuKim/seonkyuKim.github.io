---
title: "08: GitHub 시작하기"
categories: git-tutorial
tags:
- git tutorial
author_profile: true
---

여러 명의 사람들과 프로젝트를 진행하거나, 오픈 소스 프로젝트를 만들고 싶다면 서버를 만들어 공유할 수 있어야 합니다. 직접 서버를 만들 수도 있지만, 저희는 **GitHub**라는 Git 저장소 호스트를 이용하겠습니다.

GitHub은 가장 큰 Git 저장소 호스트입니다. 수백만 개발자가 모여서 수백만 프로젝트를 수행하는 중추입니다. Git 저장소를 GitHub에 만들어 운영하는 비율이 높습니다. 많은 오픈 소스 프로젝트는 GitHub을 이용해서 Git 호스팅, 이슈 트래킹, 코드 리뷰, 등등의 일을 합니다. Git을 많이 사용하다 보면 Git 프로젝트 자체에는 참여하지 않더라도 GitHub을 꼭 써야 하는 상황이 오거나 스스로 쓰고 싶어질 것입니다.

**Github의 중요성**<br><br>프로젝트 경험이 없으신 분들은 github의 중요성을 무시할 수 있습니다. 하지만 다시 한 번 말씀드립니다. 여러분들이 다른 사람들과 프로젝트를 진행하게 될 것이라면 github를 이용한 버전 관리는 필수입니다.<br>또한 github에는 무궁무진한 오픈 소스가 있습니다. 여러분들이 github를 알아야 이런 오픈소소를 사용할 수 있고, 또 여러분들의 오픈소스를 공유할 수 있습니다.
{: .notice--info}

github를 주소창에 검색하고 계정을 만듭니다. 무료 버전으로 진행하시면 됩니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-26-starting-github/01.png){: .align-center}

계정을 만드셨다면 로그인 후 좌측 sidebar에서 New 버튼을 눌러 새로운 저장소(Repository)를 만드세요. 이 저장소를 여러분의 원격 저장소(remote)로 사용할 것입니다. 저장소의 이름을 **doClone**이라 하겠습니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-26-starting-github/02.png){: .align-center}

Create repository 버튼을 누르시면 다음 화면과 같이 나올 것입니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-26-starting-github/03.png){: .align-center}

위의 주소가 여러분이 GitHub에 만든 git 저장소의 주소입니다. 혹시 03: Git 저장소 만들기의 두 번째 방법이 기억나십니까? 여러분은 제가 이렇게 만든 저장소를 위의 주소를 사용하여 clone한 것입니다.

여러분은 이제 로컬 컴퓨터에서 작업한 내용들을 GitHub에 올려 다른 사람들과 공유할 수 있습니다. 또한 GitHub를 돌아다니며 다른 사람들의 작업들을 볼 수도 있습니다. 

사실, 여러분이 보고 계신 이 블로그도 GitHub를 이용하여 만든 것입니다. 즉, 여러분은 `git clone`과 같은 명령어로 언제든 제 블로그 전체를 복사하실 수 있습니다. [https://github.com/seonkyuKim/seonkyuKim.github.io](https://github.com/seonkyuKim/seonkyuKim.github.io)로 이동하시면 제 블로그 구조, 제가 한 커밋들을 모두 보실 수 있습니다. 여러분도 [07: GitHub 시작하기](https://seonkyukim.github.io/git-tutorial/starting-github/)를 배우신 후 여러분만의 GitHub 저장소를 이용하세요!

## GitHub 둘러보기
GitHub 가입을 마치셨으면 한 번 둘러보시길 바랍니다. 다음 글을 참고하세요: [https://meetup.toast.com/posts/141](https://meetup.toast.com/posts/141)

