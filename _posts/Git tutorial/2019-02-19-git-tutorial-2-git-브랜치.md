---
title: "Git Tutorial 2: Git 브랜치"
categories: git-tutorial
tags:
- git
- tutorial
toc: true
author_profile: true
---

저번 시간에 3개의 커밋을 진행했고, github 리모트 저장소로 push하는 것 까지 진행했습니다. 이번 시간에는 커밋 히스토리를 조회하고, 브랜치를 배우면서 협업을 할 수 있게 한 걸음 나아가겠습니다. 그 전에 git을 조금 더 자유롭게 쓰기 위해 몇 가지 중요한 기능을 먼저 살펴보고 가겠습니다.


# 되돌리기
작업을 진행하다보면 항상 되돌리고 싶을 때가 있습니다. (더군다나 git의 주 목적 중 하나가 예전 버전으로 돌아가기 위함입니다.) 지금부터 실수를 해서 되돌리고 싶은 경우, 혹은 예전 버전으로 돌아가고 싶은 경우에 어떻게 해야할지 배우도록 하겠습니다.

## Modified 파일 최근 버전으로 되돌리기
이해를 위해 한 가지 상황을 가정하겠습니다. **mystory.txt** 파일에 이야기를 추가했습니다. 하지만 해당 내용이 마음에 안 들어 가장 최근 커밋 버전으로 되돌아 가고 싶습니다.

다음과 같이 파일을 수정합시다:
    
    # mybook/mystory.txt
    
    츠키라는 아이가 한국에 살았습니다. 머리가 좋지는 않았지만 매일매일 열심히 살았습니다.
    하지만 그럼에도 불구하고 잭은 좋은 대학을 가지 못했습니다.
    어느 날, 악몽을 꾸었습니다. 하늘에서 태양이 떨어져 도망도망 도망가는 꿈을 꾸었습니다.
    
이야기가 상당히 난해하군요;; 파일의 상태는 어떻게 바뀌었을까요? 지난 시간 배운 내용을 토대로 추측한 뒤 `git status`로 확인해봅시다. 혹시 예상한 것과 다른 화면이 출력되었다면 [Git Tutorial 1: Git의 기초](https://seonkyukim.github.io/git-tutorial/git-tutorial-1-git%EC%9D%98-%EA%B8%B0%EC%B4%88/#%ED%8C%8C%EC%9D%BC%EC%9D%98-%EC%83%81%ED%83%9C-%ED%99%95%EC%9D%B8%ED%95%98%EA%B8%B0git-status)를 복습하고 오시기 바랍니다.

다음과 같이 Modified 상태로 나올 것입니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-checkout-01.png){: .align-center}

이때, 바꾼 내용이 마음에 안 들어 가장 최근 버전으로 해당 파일을 되돌리고 싶습니다. 이런 경우 `git checkout -- <fileName>` 명령어를 사용하시면 됩니다(`git status` 출력 화면을 자세히 보시면 해당 명령어를 사용하라고 나와있습니다):

```shell
$ git checkout -- mystory.txt
```

이후 파일을 확인하시면 가장 최근 커밋에 저장된 내용으로 바뀌어 있습니다. 

```shell
$ cat mystory.txt    # mystory.txt 파일 내용 출력
츠키라는 아이가 한국에 살았습니다. 머리가 좋지는 않았지만 매일매일 열심히 살았습니다.
하지만 그럼에도 불구하고 잭은 좋은 대학을 가지 못했습니다.
```

**Cat이란?**<br><br>파일의 내용을 볼 수 있는 명령어입니다. `cat <fileName>`과 같이 사용합니다.
{: .notice--info}

**Note**<br><br>`git checkout -- <fileName>` 명령어는 꽤 위험하다는 것을 아셔야 합니다. 원래 파일로 덮어썼기 때문에 수정한 내용은 전부 사라집니다.
{: .notice--warning}

**To-do List**<br><br>- `git checkout -- <fileName>` 명령어 실행하기
{: .notice--success}

## Commit 수정하기

자세히 살펴보니 주인공의 이름이 전부 안 바뀌었군요... 모두 '츠키'라고 바꾸어야 하지만 한 곳은 그대로 '잭'이라고 되어있습니다. 이를 수정하기 위해서는 어떻게 해야할까요?

주인공의 이름을 다시 바꾼 후, `git add`를 한 다음 `git commit -m "change all the name of main character"`과 같이 메세지를 다시 입력해주어야 합니다.

하지만 여기서 커밋을 왜 하고 커밋 메세지를 왜 작성하는지 다시 생각해 볼 필요가 있습니다. 그 이유는 **돌아갈 수 있는 중요한 체크포인트를 기록하기 위함입니다.** 그렇다면 주인공의 이름을 바꾸었다는 것을 굳이 두 개의 커밋으로 기록할 필요가 있을까요? 이런 경우는 과감히 하나의 커밋으로 합쳐주는 것이 좋습니다. (여러분들이 나중에 코드를 작성하여 커밋할 때도, 이와 같은 습관을 들이는 것이 좋습니다.)

이렇게 다시 커밋하고 싶다면 변경 사항을 Staging Area에 추가한 다음 `--amend` 옵션을 사용하여 커밋을 재작성 할 수 있습니다. '잭'이라고 되어있는 주인공 이름을 마저 '츠키'로 바꿉시다:

    # mybook/mystory.txt
    
    츠키라는 아이가 한국에 살았습니다. 머리가 좋지는 않았지만 매일매일 열심히 살았습니다.
    하지만 그럼에도 불구하고 츠키는 좋은 대학을 가지 못했습니다.    # 수정 부분

이후 다음과 같은 명령어를 사용합시다:

```shell
$ git add mybook.txt
$ git commit --amend
```

커밋 메세지를 수정하고 싶다면 수정하셔도 됩니다. 커밋이 완료되면 다음과 같이 뜹니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-commit-amend-01.png){: .align-center}

**Note**<br><br>위의 작업은 기존 커밋을 수정하는 것이 아니라, 두 번째 커밋이 첫 번째 커밋을 완전히 덮어 쓰는 것입니다. 이전의 커밋은 일어나지 않은 일이 되는 것이고 히스토리에도 남지 않습니다.
{: .notice--warning}

**To-do List**<br><br>- `git commit --amend` 명령어 실행하기
{: .notice--success}
<br>

# 커밋 히스토리 조회하기(git log)
지금까지 했던 커밋들의 내역을 `git log`를 사용하여 확인할 수 있습니다. 다음 명령을 실행해 보세요:

```shell
$ git log
```

다음과 같이 출력될 것입니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-log-01.png){: .align-center}

가장 위에 있는 커밋이 최근 커밋이고 밑으로 갈수록 오래된 커밋입니다. 커밋의 SHA-1 체크섬, 저자 정보, 커밋 날짜, 커밋 메세지를 차례로 보여줍니다. (SHA-1 체크섬은 해당 커밋의 고유 번호라고 생각하시면 됩니다. 이를 **해시**라고 하기도 합니다.)

커밋의 몇 가지 유용한 옵션을 살펴보고자 합니다. 먼저 `-2`와 같이 숫자를 뒤에 붙여주면 최근 해당 개수의 커밋만을 보여줍니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-log-02.png){: .align-center}

`--pretty`옵션을 사용하면 원하는 포멧으로 커밋 히스토리를 조회할 수 있습니다. 다음은 `--pretty=oneline`을 추가하여 한 줄로 표기한 것입니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-log-03.png){: .align-center}

원하는 포멧을 직접 설정할 수도 있습니다. 다음은 `--pretty=format:"%H, %an`를 이용하여 커밋 해시와 저자만 나타낸 것입니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-log-04.png){: .align-center}

사용 가능한 다양한 포멧에 대해서는 [git book](https://git-scm.com/book/ko/v2/Git%EC%9D%98-%EA%B8%B0%EC%B4%88-%EC%BB%A4%EB%B0%8B-%ED%9E%88%EC%8A%A4%ED%86%A0%EB%A6%AC-%EC%A1%B0%ED%9A%8C%ED%95%98%EA%B8%B0#pretty_format)을 참조하십시오.

커밋들 간의 차이점을 보고 싶으시다면 `-p`, 혹은 `--patch` 옵션을 사용하십시오:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-log-05.png){: .align-center}

`git diff`를 사용한 것과 비슷한 화면이 출력되었습니다. 또한 터미널에 모든 내용을 다 출력되지 않았습니다. 키보드 아래 키를 눌러 전체 내용을 볼 수 있습니다. 출력을 그만 보고 싶으시다면 `q`를 누르시면 됩니다.

더 다양한 옵션에 대해서는 [git reference](https://git-scm.com/docs/git-log)를 참조하십시오.

**To-do List**<br><br>`git log` 명령어 다양한 옵션으로 실행하기
{: .notice--success}
<br>

# 브랜치(git branch)

브랜치를 이해하기 위해서는 커밋들간의 관계를 조금 살펴보아야 합니다. 각각의 커밋은 이전 커밋의 해시 정보를 포함하고 있습니다. 즉, 이전 커밋으로의 **포인터**를 갖고 있습니다. (이해가 잘 안 되시는 분들은 이전 커밋으로 갈 수 있는 연결고리가 존재한다고 생각하시면 좋을 것 같습니다.)

`git log`로 살펴보았듯이, 현재 3개의 커밋이 존재합니다. 그 3개의 커밋들간의 관계를 나타내면 다음과 같을 것입니다:

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

# Merge(git merge)

위에서 설명했듯이, 브랜치는 갈라지기도 하지만 나중에 합칠수도 있습니다. 자기소개서를 예로 들어보겠습니다. A라는 브랜치에서는 자기소개서 1번 문항을 작성하고, B라는 브랜치에서는 자기소개서 2번 문항을 작성했다고 가정합시다. 각각의 브랜치에서 해당 항목 작성이 끝난 후, 하나의 자기소개서로 합쳐야 합니다. 이 과정을 브랜치 A와 브랜치 B를 **merge**한다고 하며 git에서는 `git merge <branchName>` 명령어를 이용하면 자동적으로 합쳐줍니다. merge를 진행한 후에는 하나의 파일에 자기소개서 1번 문항과 2번 문항이 모두 작성되어 있을 것입니다. 이제 merge에 대해 더 자세히 알아보겠습니다.

## 1. 3-way-merge

merge의 종류에는 크게 두 가지가 있습니다. 먼저 저희의 경우처럼 브랜치가 갈라진 경우에 merge를 어떻게 진행하는지 알아보겠습니다.

갈라진 두 브랜치를 합쳐주기 위해서는 merge하고자 하는 두 브랜치가 가리키는 커밋들과 공통의 조상 커밋(즉, 갈라지기 시작하는 커밋)을 이용하여 merge를 진행하고, 이를 3-way-merge라 합니다. 다음 그림에서 빨간 테두리의 커밋들을 이용합니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-merge-01.png){: .align-center}

저희는 **sad** 브랜치와 **master** 브랜치를 합칠 필요가 전혀 없지만, merge를 연습하기 위해 브랜치들을 합쳐봅시다. 결국 하나의 브랜치로 합쳐지는 것은 같지만, 다른 브랜치에서 **master**브랜치를 merge하기보다는 일반적으로 **master** 브랜치에서 다른 브랜치들을 merge합니다. 저희도 **master** 브랜치에서 **sad** 브랜치를 merge합시다. 

**master** 브랜치에서 **sad** 브랜치를 merge하기 위해서는 먼저 **master** 브랜치로 이동해야 합니다. `git checkout master` 명령어를 이용하여 **master** 브랜치로 이동한 후, `git merge sad` 명령어를 사용하여 **sad** 브랜치를 merge합시다. merge를 성공적으로 진행될 경우, 다음과 같이 두 브랜치를 합친 내용의 새로운 커밋이 생기며, **master** 브랜치의 포인터가 이동해야 합니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-merge-02.png){: .align-center}

이제 다음과 같이 명령어를 사용하여 직접 해봅시다.

```shell
$ git branch
* master    # *가 표시된 브랜치가 현재 브랜치입니다.
  sad
$ git merge sad    # master 브랜치에서 sad 브랜치 merge
```
다음과 같이 충돌 메세지가 나타날 것입니다:

```
Auto-merging outline.txt
CONFLICT (content): Merge conflict in outline.txt
Automatic merge failed; fix conflicts and then commit the result.

```
이유는 생각해보면 당연합니다. **master** 브랜치에 있는 outline.txt의 내용과 **sad** 브랜치에 있는 outline.txt의 내용, 특히 다음의 빨간 글씨가 서로 상충되어 때문에 자동으로 합칠 수 없는 것입니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-merge-03.png){: .align-center}

`git status` 명령어를 사용하여 파일의 상태를 확인하면 다음과 같이 **Unmerged paths**에 **both added**로 표시됩니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-merge-04.png){: .align-center}


이런 부분은 저희가 직접 충돌을 수정해주어야 합니다. **outline.txt**파일을 열어서 수정해봅시다. 아마 다음과 같이 이상한 문자들이 추가되었을 것입니다:

```
outline.txt

머리가 좋지는 않아 열심히 살아도 성과가 안나옴.
꿈에서 만난 토끼가 6자리 숫자를 알려줌.
그 숫자로 로또가 당첨됨.
<<<<<<< HEAD
해피 앤딩
=======
쌔드 앤딩
>>>>>>> sad
````

충돌이 일어나지 않은 부분에는 아무런 변화가 없습니다. 충돌이 일어난 부분은 &lt;&lt;&lt;&lt;&lt;&lt;&lt; 와 =======, 혹은 =======와 &gt;&gt;&gt;&gt;&gt;&gt;&gt; 사이에 표시됩니다. &lt;&lt;&lt;&lt;&lt;&lt;&lt;와 ======= 사이에 표시된 내용은 merge를 실행한 브랜치에 있는 내용, 즉 **HEAD**가 가리키고 있는 내용입니다. 반대로,  =======와 &gt;&gt;&gt;&gt;&gt;&gt;&gt; 사이에 표시된 내용은 merge를 하고자 하는 브랜치, 즉 **sad** 브랜치의 내용입니다. 이 표시들을 기반으로 원하는 것을 선택하여 수정을 마친 후 충돌이 일어난 파일을 커밋해주면 merge가 완성됩니다.

저는 해피 엔딩이 좋으므로 쌔드 엔딩 부분을 지워주겠습니다:

```
outline.txt

머리가 좋지는 않아 열심히 살아도 성과가 안나옴.
꿈에서 만난 토끼가 6자리 숫자를 알려줌.
그 숫자로 로또가 당첨됨.
해피 앤딩
````

수정을 완료한 후 `git status`로 파일의 상태를 다시 확인해봅시다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-merge-05.png){: .align-center}

**Unmerged paths**에 상태가 **both added**에서 **both modified**로 바뀐 것을 확인할 수 있습니다. 이제 `git add`와 `git commit`을 이용해서 merge 마무리를 해줍시다:

```shell
$ git add outline.txt
$ git commit -m "merge fix"
[master 7bd1cbb] merge fix    # 성공적으로 수정했을 경우 출력 메세지
```

> 3-way-merge가 아무런 충돌 없이 성공적으로 일어난 경우, Merge made by the 'recursive' strategy라 표시됩니다.

## 2. fast-forward merge

fast-forward merge는 3-way-merge보다 훨씬 간단하므로 개념적으로만 설명하고 넘어가겠습니다. 

**master** 브랜치에서 다음과 같이 작업을 하고 있다고 가정합시다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-merge-06.png){: .align-center}

이때, 급하게 수정해야 할 버그가 생겨 **hotfix** 브랜치를 만들어 버그를 수정했습니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-merge-07.png){: .align-center}

이후, 수정 내역을 반영해주기 위해 **master** 브랜치에서 **hotfix** 브랜치를 merge 해줍니다. 이때는 공통의 조상을 이용해 merge하는 것이 아니기 때문에 **master** 브랜치가 앞으로 이동하게 됩니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-merge-08.png){: .align-center}

> fast-forward merge를 성공했을 경우, Fast-forward라 표시됩니다.

## git log --graph

앞선 커밋 히스토리 조회하기에서 멋진 기능 하나를 소개하지 않았습니다. 바로 `git log --graph` 입니다. 이는 커밋 히스토리를 그래프로 브랜치까지 꽤나 멋있게 보여줍니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-21-git-tutorial-2-git-브랜치/git-merge-09.png){: .align-center}
