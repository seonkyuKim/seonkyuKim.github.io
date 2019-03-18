---
title: "04: 파일 커밋하기(git add, commit)"
categories: git-tutorial
tags:
- git tutorial
author_profile: true
---
  
파일을 git 저장소에 저장해 봅시다. git을 시작한 디렉토리 안에 저장할 파일을 하나 만듭니다. 저는 **project** 디렉토리에 **sample.txt** 파일을 만들도록 하겠습니다.

**project/sample.txt** 안에 다음과 같이 내용을 입력해보겠습니다:
    
    This is sample file


>  파일 만들기에 어려움을 느끼시는 분께서는 [02: 기본적인 명령어](https://seonkyukim.github.io/git-tutorial/basic-cmd/)를 보고 오시기 바랍니다.

`git add <fileName>` 명령어를 사용하여 저장소에 저장할 수 있습니다:

``` 
$ git add sample.txt
```
<br>

아쉽게도 아직은 완벽히 저장되지 않았습니다. `git commit <fileName>` 명령어를 이용하여 '영구적인 스냅샷'인 커밋을 완료해 봅니다(이해가 안되시더라도 일단 따라해 봅시다):

``` 
$ git commit -m "first commit"
```
<br>

다음과 같이 표시된다면 첫 번째 커밋을 성공적으로 완료하신 겁니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-18-git-add-commit/01.png){: .align-center}

축하드립니다! 첫 번째 커밋을 완료했습니다. 이제 언제든지 "first commit"이 '스냅샷'으로 저장되어 언제든 해당 버전으로 돌아갈 수 있습니다.

## 커밋이란?

커밋에 대한 설명을 진행하기 위해서는 버전 관리가 무엇인지 알아야 합니다. 버전 관리 시스템은 파일 변화를 시간에 따라 기록했다가 나중에 특정 시점의 버전을 다시 꺼내올 수 있는 시스템입니다. (그리고 지금 배우시는 git이 버전 관리 시스템입니다.) '버전'이라는 용어를 어렵게 생각하실 필요 없이, 각각의 시점에 저장되는 파일들입니다. 예를 들어, 여러분이 작성한 '자기소개서_01', '자기소개서_02', '자기소개서_03' 등이 모두 하나의 '버전'인 것입니다.

git은 버전들을 관리할 때 **각 버전의 '차이점'을 저장하는 것이 아니라 버전을 저장할 때의 각각의 시점 전체를 하나의 '스냅샷'으로 저장합니다**. 이러한 각 시점의 스냅샷을 git에서 **커밋**이라 부르고, 커밋을 만드는 과정 역시 **커밋**이라 합니다. 위에서 우리는 첫 번째 **커밋**을 만든 것입니다. 커밋은 한 번 저장되면 수정할 수 없으며 영구적으로 저장됩니다. 또한 언제든지 해당 커밋으로 되돌아 올 수 있습니다. (왜냐하면 하나의 시점 전체를 저장한 것이기 때문이죠!) 

자세한 설명은 git book '[1.1시작하기-버전 관리란?](https://git-scm.com/book/ko/v2/%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-%EB%B2%84%EC%A0%84-%EA%B4%80%EB%A6%AC%EB%9E%80%3F)'와 '[1.3시작하기-Git 기초](https://git-scm.com/book/ko/v2/%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-Git-%EA%B8%B0%EC%B4%88)'의 내용을 참고하시기 바랍니다.



## Working directory, Staging area, .git directory


커밋을 만들기 위해 왜 **add**와 **commit**, 총 두 번의 과정을 거쳐야 하는지 궁금하실 겁니다. 사실, 이 부분은 [git book](https://git-scm.com/book/ko/v2)의 '[1.3 Git의 기초](https://git-scm.com/book/ko/v2/%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-Git-%EA%B8%B0%EC%B4%88)'에 기술되어 있지만 중요하기 때문에 한 번 더 다루고자 합니다. 

git에는 **크게 3가지 공간**이 있습니다. 작업을 하고 있는 로컬 컴퓨터의 공간인 '**워킹 디렉토리**(*working directory*)', 커밋할 파일에 대한 정보를 저장하는 '**스테이징 에어리어**(*staging area*)', 스테이징 에어리어의 파일들을 커밋하여 영구적인 스냅샷을 저장하는 장소인 '**Git 디렉토리**(.*git directory*)'가 있습니다. 

위에서 **sample.txt** 파일을 만들었을 때, 파일이 저장되는 로컬 컴퓨터의 공간이 **워킹 디렉토리**입니다. **sample.txt** 파일을 처음 만들게 된다면 다음과 같이 워킹 디렉토리에 존재할 것입니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-18-git-add-commit/02.png){: .align-center}

이제 이 파일을 영구적으로 저장하기 위해서는 먼저 **스테이징 에어리어**에 저장하고 싶은 파일들의 정보를 등록해주어야 합니다.  `git add <fileName>` 명령어를 사용하여 자기소개서를 스테이징 에어리어에 올려줍시다. 이때, 워킹 디렉토리의 파일을 **옮기는 것이 아니라 복사**한다고 생각해야합니다. 즉, 워킹 디렉토리의 파일을 수정해도, 스테이징 에어리어의 파일에는 반영되지 않습니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-18-git-add-commit/03.png){: .align-center}

>`git add .` 명령어를 사용하시면 현재 폴더 안의 모든 파일들을 add할 수 있습니다.

이제 **스테이징 에어리어에 있는 파일들을 가지고** 최종적으로 Git 저장소에 저장하여 영구적인 스냅샷으로 저장합시다. `git commit <fileName>` 명령어를 **커밋**이 완료됩니다.:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-18-git-add-commit/04.png){: .align-center}


하나의 그림으로 나타내면 다음과 같습니다:

<figure class="align-center">
<img src="{{ site.url }}{{ site.baseurl }}/assets/images/2019-02-18-git-add-commit/05.png" alt="">
<figcaption>(사진: <a href="https://git-scm.com/book/ko/v2/%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-Git-%EA%B8%B0%EC%B4%88">https://git-scm.com/book/ko/v2/시작하기-Git-기초</a>)</figcaption>
</figure> 


앞에서 저희가 했던 것을 다시 생각해봅시다. 저희는  `git add sample.txt` 명령어를 이용하여 **sample.txt** 파일에 대한 정보를 staging area에 저장한 후 `git commit sample.txt` 명령어를 이용하여 **.git** 디렉토리에 커밋을 완료한 것입니다.

**Staging Area의 위치**<br><br>사실, 스테이징 에어리어의 정보들도 '**.git**' 폴더 안에 있습니다. 위 세 가지 공간 구분은 물리적 구분이 아닌 공간의 성격에 따른 추상적인 구분이라 생각하시면 됩니다.
{: .notice--info}


## 커밋 메세지(git commit -m)

커밋을 할 때는 해당 커밋을 설명하는 메세지를 같이 저장해야 합니다. 계속해서 커밋을 만들게 되면, 변경 사항이 무엇인지 헷갈릴 것입니다. 이때, 그것을 기록하기 위한 용도로 해당 커밋을 설명하는 메세지를 작성해야 합니다.  

위에서 저희는 다음과 같은 명령어로 커밋을 완료했습니다:

``` 
$ git commit -m "first commit"
```

이때, `-m "fisrt commit"`의 의미는 커밋 메세지로 "first commit"를 저장하라는 의미입니다.

만일 `-m` 옵션 없이 `git commit sample.txt`만 입력했을 경우 다음과 같이 커밋 메세지를 작성할 수 있는 창이 뜹니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-18-git-add-commit/06.png){: .align-center}

당황하지 마시고 INSERT모드로 바꾸신 후 메세지를 첫 줄에 입력하고 저장하시면 됩니다. **만약 아무런 메세지를 입력하지 않을 경우 커밋되지 않습니다.**

### 커밋 메세지 가이드라인

git에는 [커밋 메세지 가이드라인](https://gist.github.com/robertpainsi/b632364184e70900af4ab688decf6f53)이 있습니다. 가이드라인은 다음과 같습니다:

>영문 기준입니다.

- 제목과 본문은 한 줄로 띄어 분리하십시오
- 제목을 온점으로 끝내지 마십시오
- 제목과 각 문단의 첫 글자는 대문자로 쓰십시오
- 제목은 명령조로 쓰십시오
- 한 문단은 72글자 이내로 작성하십시오
- 본문에 무엇을 했고 왜 했는지 적으십시오. 대부분의 경우 변화가 일어난 세부 사항은 생략할 수 있습니다.

커밋 메세지 예는 다음과 같습니다:

```
Short (72 chars or less) summary

More detailed explanatory text. Wrap it to 72 characters. The blank
line separating the summary from the body is critical (unless you omit
the body entirely).

Write your commit message in the imperative: "Fix bug" and not "Fixed
bug" or "Fixes bug." This convention matches up with commit messages
generated by commands like git merge and git revert.

Further paragraphs come after blank lines.

- Bullet points are okay, too.
- Typically a hyphen or asterisk is used for the bullet, followed by a
single space. Use a hanging indent.
```
<br>


## 커밋 한 번에 진행하기(git commit -a)

`git commit -a` 옵션을 사용할 경우 `git add`의 과정을 거치지 않고 바로 커밋을 할 수 있습니다. 

``` 
$ git commit -a -m "first commit"
```

**Note**<br><br>편리하다고 생각하실 수 있지만, 이는 저장하지 말아야 할 변경사항까지 모두 추가할 가능성이 있습니다. 따라서 이 기능을 사용하실 때는 주의하시기 바랍니다. 
{: .notice--warning}


## 커밋 수정하기(git commit --amend)

프로젝트를 진행하게 되면 계속해서 커밋을 하게 됩니다. 이때 자잘한 부분의 수정을 까먹고 다시 커밋을 만드는 경우가 굉장히 많습이다. 예를 들어 저는 커밋 메세지로 "fix", "fix again", "fix again...x2", "please fix!"와 같이 커밋 메세지를 만들었었습니다. 

하지만 여기서 커밋을 왜 하고 커밋 메세지를 왜 작성하는지 다시 생각해 볼 필요가 있습니다. 그 이유는 **변경한 체크포인트를 기록하기 위함입니다.** 그렇다면 자잘한 변경사항들을 굳이 여러 개의 커밋으로 기록할 필요가 있을까요? 이런 경우는 과감히 하나의 커밋으로 합쳐주는 것이 좋습니다.

이렇게 다시 커밋하고 수정하고 싶은 경우 **변경 사항을 add한 후, `--amend` 옵션과 함께 다음 커밋을 해주시면 됩니다.** 커밋 메세지를 수정하고 싶은 경우에도 이 옵션을 사용합니다.

``` 
$ git add <changedFiles>
$ git commit --amend
```


**Note**<br><br>커밋은 사실 수정이 불가능합니다. 위의 작업은 기존 커밋을 수정하는 것이 아니라, 두 번째 커밋이 첫 번째 커밋을 완전히 덮어 쓰는 것입니다. 이전의 커밋은 일어나지 않은 일이 되는 것이고 히스토리에도 남지 않습니다.
{: .notice--warning}

