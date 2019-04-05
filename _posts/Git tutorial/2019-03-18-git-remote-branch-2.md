---
title: "16: 리모트 브랜치 만들기"
categories: git-tutorial
tags:
- git tutorial
author_profile: true
---

저번 시간에는 리모트 브랜치, 트래킹 브랜치들의 개념을 살펴보았습니다. 이번에는 각각의 브랜치를 어떻게 만드는지 알아보도록 하겠습니다.

## 리모트 브랜치 만들기

로컬에서 생성한 브랜치들은 `git push`를 할 경우 자동으로 리모트 저장소로 push 되지 않습니다. 다음과 같이 명시적으로 코드를 작성해주어야 합니다:

    $ git push <remote> <localBranch>

위는 리모트 브랜치로 해당 로컬 브랜치를 push합니다. 리모트 브랜치에서는 로컬 브랜치와 다른 이름을 쓰고 싶다면 다음과 같이 명령어를 작성해야 합니다:

    $ git push <remote> <localBranch>:<branchName>

## 트래킹 브랜치 만들기

트래킹 브랜치를 만드는 방법에는 여러 가지가 있습니다. 다음 방법들은 브랜치의 이름과 관련되어 있을 뿐, 본질적으로 같다고 생각하시면 됩니다. 예시를 들어가며 설명하겠습니다.

1.**origin/serverfix**라는 리모트 브랜치를 추적하는 **iss10** 이름의 트래킹 브랜치 만들기

```
$ git checkout -b iss10 origin/serverfix
Branch 'iss10' set up to track remote branch 'serverfix' from 'origin'.
Switched to a new branch 'iss10'
```
<br>

2.리모트 브랜치와 같은 이름의 트래킹 브랜치 만들기

```
$ git checkout --track origin/serverfix
Branch 'serverfix' set up to track remote branch 'serverfix' from 'origin'.
Switched to a new branch 'serverfix'
```
<br>

3.위 방법 shortcut

만약 입력한 브랜치가 있는 리모트가 딱 하나 있고, 로컬에는 없으면 다음과 같이 줄여 쓸 수 있습니다:

```
$ git checkout serverfix
Branch 'serverfix' set up to track remote branch 'serverfix' from 'origin'.
Switched to a new branch 'serverfix'
```
<br>

4.이미 로컬에 존재하는 브랜치가 리모트 브랜치를 추적하게 하기

`-u` 혹은 `--set-upstream-to` 옵션을 사용하시면 됩니다.

```
$ git branch -u origin/serverfix
```
<br>

## 트래킹 브랜치와 리모트 브랜치 비교하기

로컬로 가져온 리모트 브랜치는 fetch를 하지 않고서는 수정할 수 없다고 배웠습니다. 따라서 트래킹 브랜치를 만들어 로컬에서 작업을 진행합니다. 이때, 리모트 브랜치와 트래킹 브랜치의 커밋 히스토리를 다음과 같이 비교할 수 있습니다. (비교는 가장 최근 fetch와 비교합니다)

```
$ git branch -vv
iss53     7e424c3 [origin/iss53: ahead 2] forgot the brackets
master    1ae2a45 [origin/master] deploying index fix
* serverfix f8674d9 [teamone/server-fix-good: ahead 3, behind 1] this should do it
testing   5ea463a trying something new
```

왼쪽에 표시된 내용은 로컬 브랜치의 이름입니다. 그 옆에는 해당 브랜치가 가리키고 있는 커밋입니다. [ ] 사이에 표시된 내용은 upstream 브랜치입니다. 그 옆에는 커밋 메세지가 표시됩니다.

[ ] 사이에 ahead는 로컬 브랜가 리모트 브랜치보다 커밋이 두 개가 앞서있다는 뜻입니다. (즉, fetch를 한 후 두 개의 커밋을 새로 만들었다고 볼 수 있습니다.) behind라고 표시되면 반대로 뒤쳐져 있다는 뜻입니다. 만약 ahead와 behind 두 개가 동시에 표시된다면, 이는 브랜치가 갈라졌다는 의미입니다. 즉, 영희의 master 브랜치는 ahead 2, behind 2 라고 표기될 것입니다.

```
# 영희의 로컬 저장소
$ git branch -vv
master     C5' [origin/master: ahead 2, behind 2] <Commit message>
```

주의할 점은 리모트 저장소와의 비교를 마지막 fetch를 기준으로 진행합니다. 따라서 꼭 fetch를 한 후 비교해주시기 바랍니다. 따라서 다음과 같이 사용할 수 있습니다:

```
$ git fetch -all; git branch -vv
```

## 브랜치 합치기

영희와 같이 트래킹 브랜치와 upstream 브랜치가 갈라졌을 경우, 두 브랜치를 앞에서 배운대로 merge해주시면 됩니다:

```
$ git merge
```

혹은 `git pull` 명령어를 사용할 수도 있습니다. 사실, 앞에서 배운 `git pull`의 의미는 fetch와 merge를 동시에 진행하는 것입니다. 일반적으로 fetch와 merge 명령을 명시적으로 사용하는 것이 pull 명령으로 한 번에 하는 것보다 좋습니다.
