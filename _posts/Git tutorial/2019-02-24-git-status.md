---
title: "07: 파일 상태 확인하기(git status)"
categories: git-tutorial
tags:
- git tutorial
author_profile: true
---

<br><br><br><br><br><br><br>

## 블로그가 이전되었습니다 => [가나비](https://gonna-be.tistory.com/21)

<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>

이번에는 git에 의해 관리되는 파일들의 가능한 **상태**(status)에 대해 알아보도록 하겠습니다. **상태**라는 말을 너무 어렵게 생각하실 필요가 없습니다. 앞선 강의에서 파일이 **스테이징 에어리어**를 거쳐 커밋되는 과정을 배웠습니다. 이때 **스테이징 에어리어**에 등록된 파일의 상태는 Staged, 커밋된 파일의 상태는 Unmodified라고 하는데, 이와 같이 각 단계에서 가능한 상태들이 존재합니다. 앞으로 각 상태에 대해 자세히 알아보겠습니다.


**Note**<br><br>앞에서는 'git 공간'의 분류를 세 가지로 했다면, 지금은 '**파일의 상태**'를 분류하는 것입니다.
{: .notice--warning}

<figure class="align-center">
<img src="{{ site.url }}{{ site.baseurl }}/assets/images/2019-02-24-git-status/04.png" alt="">
<figcaption>(사진: <a href="https://git-scm.com/book/ko/v2/Git%EC%9D%98-%EA%B8%B0%EC%B4%88-%EC%88%98%EC%A0%95%ED%95%98%EA%B3%A0-%EC%A0%80%EC%9E%A5%EC%86%8C%EC%97%90-%EC%A0%80%EC%9E%A5%ED%95%98%EA%B8%B0">https://git-scm.com/book/ko/v2/Git의-기초-수정하고-저장소에-저장하기</a>)</figcaption>
</figure> 

**워킹 디렉토리의 파일은 먼저 크게 Untracked, Tracked의 두 가지 상태로 나뉩니다.**(워킹 디렉토리는 작업중인 로컬 컴퓨터의 공간입니다.) 파일에 수정이 일어나면 git이 파일의 변경을 감지하여 사용자에게 알려주는 것과 같이 파일을 추적하는 상태를 **Tracked** 상태라고 합니다. 반대로, 파일을 저장소에 저장할 필요가 없어 git이 신경쓰지 않아도 되는 상태를 **Untracked** 상태라고 합니다.

파일을 새로 만들 경우 Untracked 상태 즉, git이 파일을 추적하지 않는 상태가 됩니다. git 저장소에 저장할 필요가 없는 파일들은 Untracked 상태로 두시면 됩니다. 이후 `git add` 명령어를 이용하여 파일을 add해주면, 앞에서 배운 것처럼 해당 파일은 staging area에 저장되어 Tracked 상태 즉, git이 파일을 추적하는 상태가 됩니다.

**Tracked 상태의 파일들은 다시 크게 Unmodified, Modified, Staged 3개의 상태로 나뉩니다.** staging area에 있는 파일들의 상태는 **Staged**입니다. staging area에 있는 파일들을 커밋하게 되면 해당 파일들은 하나의 커밋으로 저장된 후, 파일의 상태는 **Unmodified**로 내려오게 됩니다. Unmodified 상태의 파일들을 수정하게 되면 **Modified** 상태가 됩니다. 이후 다시 `git add` 명령어를 이용하여 Staged 상태로 올려준 후 커밋을 하는 과정을 반복하게 됩니다. 다시 한 번 위의 그림을 보시면 이해가 쉬울 것입니다.

**Note**<br><br>한 번 `git add`를 해주시면 직접 `git rm --cached <fileName>` 명령어를 이용하지 않는 이상 Untracked 상태가 되지 않습니다. `git rm`에 대해서는 나중에 다시 다룹니다.
{: .notice--warning}


## Untracked

`git status` 명령어를 실행했을 때 다음과 같은 화면에서 **Untracked files** 명단에 있는 빨간색 파일들이 Untracked 상태의 파일들입니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-24-git-status/01.png){: .align-center}

>git을 시작한 폴더에서 **untracked.txt**파일을 새로 만들고 `git status` 명령어를 실행한 화면입니다.


## Staged

`git status` 명령어를 실행했을 때 다음과 같은 화면에서 **Changes to be committed** 명단에 있는 초록색 파일들이 Staged 상태의 파일들입니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-24-git-status/02.png){: .align-center}

>**tracked.txt** 파일을 만들어 `git add`를 한 후 `git status` 명령어를 실행한 화면입니다.

## Modified

`git status` 명령어를 실행했을 때 다음과 같은 화면에서 **Changes not staged for commit** 명단에 있는 빨간색 파일들이 Modified 상태의 파일들입니다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-02-24-git-status/03.png){: .align-center}

**하나의 파일이 두 개의 상태가 가능한가요?**<br><br>위의 화면을 보시면 **tracked.txt**파일이 Staged 상태에도 있고 Modified 상태에도 있는 것을 확인할 수 있습니다. [04: 파일 커밋하기(git add, commit)](https://seonkyukim.github.io/git-tutorial/git-add-commit/)에서 Staging Area에 있는 파일은 워킹 디렉토리의 파일이 **옮겨지는 것이 아니라 복사되는 것**이라는 점 기억하시나요? 그렇기 때문에 `git add`를 한 후 그 파일을 수정하게 되면 Staging Area에 있는 파일에는 수정내용이 반영되지 않습니다. 따라서 하나의 파일이 위와 같은 두 개의 상태를 가질 수 있습니다.
{: .notice--info}

## Unmodified

성공적으로 커밋을 진행한 후에 `git status` 명령어를 실행하면 Unmodified 상태의 파일은 화면에 출력되지 않습니다. 다만 주의하실 점은 `git commit` 명령어는 Staging Area의 파일들을 커밋한다는 점입니다. 따라서 `git add`를 한 후 그 파일을 수정한 경우, 다시 `git add`를 이용해 변경 파일을 Staging Area에 올려주지 않고 `git commit`을 진행한다면 변경 이전의 파일이 커밋됩니다.

## Modified 파일 되돌리기(git checkout -- <file>)

워킹 디렉토리의 modified 특정 파일을 가장 최근 커밋 버전으로 되돌릴 수 있습니다. 즉, 워킹 디렉토리에서의 수정 사항을 제거합니다. 하지만 이 기능은 되돌릴 수 없으니 주의해 주시기 바랍니다. 다음과 같이 **sample.txt** 파일을 가장 최근 커밋 버전으로 되돌릴 수 있습니다:

```
$ git checkout -- sample.txt
```




