---
title: "09: git 브랜치(git branch)"
categories: git-tutorial
tags:
- git
- tutorial
author_profile: true
---

브랜치를 이해하기 위해서는 커밋들간의 관계를 조금 살펴보아야 합니다. 각각의 커밋은 이전 커밋의 해시 정보를 포함하고 있습니다. 즉, 이전 커밋으로의 **포인터**를 갖고 있습니다. (이해가 잘 안 되시는 분들은 이전 커밋으로 갈 수 있는 연결고리가 존재한다고 생각하시면 좋을 것 같습니다.)

현재 3개의 커밋이 존재합니다. 그 3개의 커밋들간의 관계를 나타내면 다음과 같을 것입니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-branch-01.png){: .align-center}

초록색 박스는 각각의 커밋을 나타내고 쓰여있는 번호는 각 커밋의 해시 앞 5자리 입니다. 각각의 커밋은 해당 시점의 스냅샷을 기록하고 있습니다. 가장 최근 커밋은 오른쪽 `f7392` 커밋입니다. 커밋을 추가할 때마다 같은 방식으로 계속해서 추가되며 연결이 길어집니다. 이 모습이 마치 나무와 같아 **브랜치**라 부릅니다.(비록 아직 나뭇가지 없이 기둥 하나만 있지만요.)

이와 같이 처음 생기는 브랜치를 **Master** 브랜치라 부릅니다. 이제 여기서 나무가 뻗어 나가듯이 브랜치를 추가할 수 있습니다. 브랜치를 추가하는 이유는 대학별 자기소개서를 따로 쓰듯이, 프로젝트가 다른 방향으로 나가게 될 경우 추가해주기도 합니다. 또는 프로젝트에서 다른 기능을 추가할 때 사용하기도 합니다.

예를 들어 웹페이지를 제작한다고 할 때 한 명은 로그인 기능을, 다른 한 명은 게시판 기능을 맡았다고 가정합시다. 이때, 두 명이 모두 Master 브랜치에서 작업할 경우 서로 자신의 코드를 push를 하여 Master 브랜치의 코드는 엉키고 말 겁니다. 그래서 로그인 브랜치, 게시판 브랜치를 따로따로 만들어 해당 공간에서 작업을 진행하고 나중에 합치는 과정을 거칩니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-branch-04.png){: .align-center}

혹시 이해가 잘 안되신다면 다음 글을 읽어보시기 바랍니다: [누구나 쉽게 이해할 수 있는 Git 입문](https://backlog.com/git-tutorial/kr/stepup/stepup1_1.html)


## 브랜치 만들기(git branch)

실제로 브랜치를 만들어 봅시다. 연습을 위한 것이므로 기존 이야기 개요에서 sad ending으로 이야기를 진행할 브랜치를 만든다고 가정합시다.

다음과 같이 `git branch <branchName>` 명령어를 사용하여 **sad** 브랜치를 만듭니다:

```shell
$ git branch sad
```

`git branch` 명령어로 어떤 브랜치가 있는지 확인할 수 있습니다:

```shell
$ git branch
* master
sad
```

새롭게 만들어진 브랜치는 그 시점의 가장 최근 커밋을 가리키고 있습니다. &#42; 표시가 있는 브랜치가 현재 있는 브랜치입니다. 이와 같이 현재 브랜치를 가리키고 있는 포인터를 **HEAD**라고 합니다. 다음 그림으로 보시면 이해가 편할 것입니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-branch-02.png){: .align-center}

## 브랜치 이동하기(git checkout)

**sad**브랜치로 이동하여 개요를 수정합시다. `git checkout <branchName>` 명령어를 이용하면 됩니다:

```shell
$ git checkout sad
Switched to branch 'sad'   # 성공적으로 checkout 했을 경우 출력 메세지
```

해당 명령어를 이용하면 **master** 브랜치를 가리키고 있던 **HEAD**가 **sad** 브랜치로 이동하게 됩니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-branch-03.png){: .align-center}

이동한 브랜치에서 개요를 바꾼 뒤 커밋을 해 봅시다. 

# mybook/outline.txt in sad branch

머리가 좋지는 않아 열심히 살아도 성과가 안나옴.
꿈에서 만난 토끼가 6자리 숫자를 알려줌.
그 숫자로 로또가 당첨됨.
쌔드 앤딩

```shell
$ git add outline.txt
$ git commit -m "sad ending"
```

브랜치의 모습이 다음과 같습니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-branch-05.png){: .align-center}

**master** 브랜치에서는 해피 앤딩으로 이야기를 끝냅시다. **master** 브랜치로 다시 이동한 후 개요를 수정한 뒤 커밋해주세요.

```shell
$ git checkout master
Switched to branch 'master'    # 성공적으로 checkout 했을 경우 출력 메세지
```

# mybook/outline.txt in master branch
머리가 좋지는 않아 열심히 살아도 성과가 안나옴.
꿈에서 만난 토끼가 6자리 숫자를 알려줌.
그 숫자로 로또가 당첨됨.
해피 앤딩

```shell
$ git add outline.txt
$ git commit -m "happy ending"
```

이제 브랜치의 모습이 다음과 같습니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-branch-06.png){: .align-center}
