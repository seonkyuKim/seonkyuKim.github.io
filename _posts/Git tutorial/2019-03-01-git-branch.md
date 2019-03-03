---
title: "09: 브랜치 시작하기(git branch)"
categories: git-tutorial
tags:
- git
- tutorial
author_profile: true
---

브랜치를 이해하기 위해서는 커밋들간의 관계를 조금 살펴보아야 합니다. 각각의 커밋은 이전 커밋의 해시 정보를 포함하고 있습니다. 즉, 이전 커밋으로의 **포인터**를 갖고 있습니다. (이해가 잘 안 되시는 분들은 이전 커밋으로 갈 수 있는 연결고리가 존재한다고 생각하시면 좋을 것 같습니다.)

`git log`를 통해 확인한 제 디렉토리의 커밋들입니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-03-01-git-branch/01.png){: .align-center}

위 3개의 커밋들간의 관계를 나타내면 다음과 같습니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-03-01-git-branch/02.png){: .align-center}

초록색 박스는 각각의 커밋을 나타내고 쓰여있는 번호는 각 커밋의 해시 앞 5자리 입니다. 각각의 커밋은 해당 시점의 스냅샷을 기록하고 있습니다. 가장 최근 커밋은 오른쪽 **e2136** 커밋입니다. 커밋을 추가할 때마다 같은 방식으로 계속해서 추가되며 연결이 길어집니다. 이 모습이 마치 나무와 같아 **브랜치**라 부릅니다.(비록 아직 나뭇가지 없이 기둥 하나만 있지만요.)

이와 같이 **처음 생기는 브랜치**를 **master** 브랜치라 부릅니다. 이제 여기서 나무가 뻗어 나가듯이 브랜치를 추가할 수 있습니다. 브랜치를 추가하는 이유는 대학별 자기소개서를 따로 쓰듯이, 프로젝트가 다른 방향으로 나가게 될 경우에 사용합니다. 또는 프로젝트에서 다른 기능을 추가할 때 별도의 공간에서 작업하기 위해 사용하기도 합니다.

예를 들어 웹페이지를 제작한다고 할 때 한 명은 로그인 기능을, 다른 한 명은 게시판 기능을 맡았다고 가정합시다. 이때, 두 명이 모두 **master** 브랜치에서 작업할 경우 서로 자신의 코드를 push를 하면 **master** 브랜치의 코드는 엉키고 말 겁니다. 그래서 로그인 브랜치, 게시판 브랜치를 따로따로 만들어 해당 공간에서 작업을 진행하고 나중에 합치는 과정을 거칩니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-03-01-git-branch/03.png){: .align-center}

혹시 이해가 잘 안되신다면 다음 글을 읽어보시기 바랍니다: [누구나 쉽게 이해할 수 있는 Git 입문](https://backlog.com/git-tutorial/kr/stepup/stepup1_1.html)

## master branch

기본적으로 만들어지는 브랜치는 **master** 브랜치이고, 여기에 다른 브랜치들을 추가할 수 있습니다. 각각의 **&lt;branch&gt;**는 **해당 브랜치의 마지막 커밋들을 가리키고 있습니다**. 위에서 저는 3개의 커밋을 만들었습니다. 각각의 과정을 그림으로 보면서 **master** 브랜치가 가리키고 있는 커밋의 변화를 확인해보세요:

- 첫 번째 커밋 후:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-03-01-git-branch/04.png){: .align-center}

- 두 번째 커밋 후:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-03-01-git-branch/05.png){: .align-center}

- 세 번째 커밋 후:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-03-01-git-branch/06.png){: .align-center}

## 브랜치 만들기(git branch)

실제로 브랜치를 만들어 봅시다. 자기소개서 3번 문항까지 완료한 상태에서, 대학마다 다른 4번 문항을 작성하기 위해 브랜치를 만든다고 합시다.

다음과 같이 `git branch <branchName>` 명령어를 사용하여 서울대 자기소개서를 작성할 **snu** 브랜치를 만듭니다:

```shell
$ git branch snu
```

`git branch` 명령어로 어떤 브랜치가 있는지 확인할 수 있습니다:

```shell
$ git branch
* master
snu
```

이렇게 만든 새 브랜치는 작업하고 있던 마지막 커밋을 가리킵니다. &#42; 표시가 있는 브랜치가 현재 있는 브랜치입니다. 이와 같이 **현재 브랜치를 가리키고 있는 포인터**를 **HEAD**라고 합니다. 다음 그림으로 보십시오:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-03-01-git-branch/07.png){: .align-center}

## 브랜치 이동하기(git checkout)

**snu** 브랜치로 이동하여 자기소개서를 작성합시다. `git checkout <branchName>` 명령어를 이용하면 됩니다:

```shell
$ git checkout snu
Switched to branch 'snu'   # 성공적으로 checkout 했을 경우 출력 메세지
```

**master** 브랜치에서 **sad** 브랜치로 이동했기 때문에 이제 **HEAD**가 **sad** 브랜치를 가리키고 있습니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-03-01-git-branch/08.png){: .align-center}


**브랜치 생성과 함께 이동하기**<br><br>브랜치 생성과 이동을 한 번에 할 수 있습니다. `git checkout -b snu`와 같이 사용해주시면 브랜치를 만들고 해당 브랜치로 이동합니다.
{: .notice--info}

## 갈리지는 브랜치

**snu** 브랜치로 이동해서 자기소개서 작성을 완료한 다음 커밋해줍시다. 그럼 다음과 같은 모습이 됩니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-03-01-git-branch/09.png){: .align-center}

`git checkout master` 명령어로 **master** 브랜치로 돌아온 후 자기소개서의 내용을 확인해보면 4번 문항을 적기 이전인 **e2136** 커밋의 상태인 것을 확인할 수 있습니다. **master** 브랜치에서 지스트 대학 자기소개서 4번 문항을 작성한 다음 커밋을 완료합시다. 다음과 같은 모습이 됩니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-03-01-git-branch/10.png){: .align-center}

이제 여러분은 `chekcout`, `add`, `commit`만을 이용하여 브랜치들을 이동하며 독립적으로 작업을 진행할 수 있습니다.
