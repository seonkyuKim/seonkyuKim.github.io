---
title: "07: 리모트 저장소(git remote)"
categories: git-tutorial
tags:
- git
- tutorial
author_profile: true
---

**리모트 저장소는 인터넷이나 네트워크 어딘가에 있는 저장소를 말합니다.** 예를 들어 github에 만든 저장소 역시 리모트 저장소입니다.

그런데 인터넷에는 저희가 사용하고자 하는 저장소 이외에도 다른 사람들이 만든 많은 어마어마한 수의 저장소가 있습니다. 그렇다면 파일을 원하는 저장소에 보내기 위해서는 어떻게 해야할까요? **특정 경로의 저장소에 파일을 보내라고 git에게 알려주어야 합니다.** git은 이를 편리하게 하기 위해 리모트 저장소의 이름과 함께 해당 저장소로의 경로를 저장하고 있습니다. 

리모트 저장소를 관리할 줄 알아야 다른 사람과 함께 일할 수 있습니다. 저장소는 여러 개가 있을 수 있는데 어떤 저장소는 읽고 쓰기 모두 할 수 있고 어떤 저장소는 읽기만 가능할 수 있습니다. 다른 사람들과 함께 일한다는 것은 리모트 저장소를 관리하면서 데이터를 보내고 받는 것을 반복하는 것입니다.

## 리모트 저장소 확인하기(git remote)

`git clone`을 이용하여 git을 시작한 경우, 기본적으로 **origin**이라는 리모트 저장소를 갖고 있습니다. 이 리모트 저장소는 처음에 clone한 저장소입니다.
> `git init`으로 git을 시작한 경우 아무런 리모트 저장소를 갖고 있지 않습니다.

다음과 같은 명령어로 등록된 리모트 저장소 목록을 확인할 수 있습니다:

```shell
$ git remote
origin
```

리모트 저장소의 이름과 해당 저장소의 경로를 확인하고 싶다면 `-v` 옵션을 추가해주어야 합니다:

```shell
$ git remote -v
origin    https://github.com/seonkyuKim/doClone.git (fetch)
origin    https://github.com/seonkyuKim/doClone.git (push)
```



## 리모트 저장소 추가하기(git remote add)

`git remote add <remoteName> <url>`명령어를 이용하여 해당 &lt;url&gt;의 리모트 저장소를 쉽게 등록할 수 있습니다. &lt;remoteName&gt;은 사용할 리모트 저장소의 이름입니다. 예시를 보십시오:

```shell
$ git remote add origin https://github.com/seonkyuKim/doClone.git
```

## 리모트 저장소에 push하기(git push)

로컬 디렉토리에 저장된 커밋을 리모트 저장소에 옮기는 것을 **push**한다고 합니다. (반대로 리모트 저장소에 있는 내용을 가져오는 것을 **pull**한다고 합니다.) [git book](https://git-scm.com/book/ko/v2) "[1.1 시작하기 - 버전관리란?](https://git-scm.com/book/ko/v2/%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-%EB%B2%84%EC%A0%84-%EA%B4%80%EB%A6%AC%EB%9E%80%3F)"에 나온 내용을 잠깐 다시 보고자 합니다.

<figure class="align-center">
<img src="{{ site.url }}{{ site.baseurl }}/assets/images/2019-2-20-git-기초/Untitled-f1197653-b761-429e-81a7-84e77c699821.png" alt="">
<figcaption>(사진: <a href="https://git-scm.com/book/ko/v2/%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-%EB%B2%84%EC%A0%84-%EA%B4%80%EB%A6%AC%EB%9E%80%3F">https://git-scm.com/book/ko/v2/시작하기-버전-관리란%3F</a>)</figcaption>
</figure> 

DVCS(분산 버전 관리 시스템)은 단순히 파일의 마지막 커밋만을 저장하지 않습니다. 그냥 저장소를 히스토리와 더불어 전부 복제합니다. 

로컬 저장소에 있는 모든 커밋을 리모트 저장소로 보내기 위해서는 `git push <remoteName> <branchName>` 명령어를 사용하면 됩니다. 반대로 리모트 저장소의 커밋을 로컬 저장소에 가져오기 위해서는 `git pull <remoteName> <branchName>` 명령어를 사용하시면 됩니다. 브랜치를 모르시는 분들은 기본적으로 master에 있다는 점만 기억하시고 다음과 같이 사용하시면 됩니다:


```shell
# push
$ git push origin master

# pull
$ git pull origin master
```
