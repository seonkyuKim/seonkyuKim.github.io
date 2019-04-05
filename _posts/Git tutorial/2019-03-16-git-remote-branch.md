---
title: "15: 리모트 브랜치 개념"
categories: git-tutorial
tags:
- git tutorial
author_profile: true
---

저번 시간에 영희의 로컬 저장소가 다음과 같이 변한 것을 확인했습니다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-03-16-git-remote-branch/01.png){: .align-center}

이때, 위 사진에서 **origin/master를 리모트 브랜치**라고 합니다. 리모트 브랜치는 `git fetch` 명령어를 통해 로컬 저장소로 가져올 수 있습니다.

리모트 저장소를 여러 개 저장할 수 있는 것처럼, 리모트 브랜치 역시 여러 개 저장할 수 있습니다. 이해를 돕기 위해 **teamA**라는 리모트 저장소를 하나 더 만들고, 해당 리모트 브랜치를 `git fetch`해보도록 하겠습니다. 다음은 **teamA**라는 리모트 저장소와, 그것을 fetch한 영희의 로컬 저장소의 모습입니다:

- 리모트 저장소
![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-03-16-git-remote-branch/02.png){: .align-center}

- 영희 로컬 저장소

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-03-16-git-remote-branch/03.png){: .align-center}

teamA의 리모트 저장소의 커밋 히스토리가 영희의 커밋 히스토리안에 포함되므로 단순히 리모트 브랜치 하나가 더 생기게 됩니다.


## 리모트 브랜치, 리모트 트래킹 브랜치, Upstream 브랜치

리모트 브랜치는 위에서 살펴본 **origin/master** 혹은 **teamA/master**입니다. 이들은 앞서 배운 로컬 브랜치들과 달리 수정을 할 수 없습니다. 단지 해당 커밋을 가리키고 있는 포인터입니다.

리모트 브랜치를 사용하기 위해서는 리모트 브랜치를 추적(tracking)하고 있는 **트래킹 브랜치**를 로컬에서 만들어 사용합니다. 그리고 트래킹 브랜치가 가리키고 있는 리모트 브랜치를 **upstream 브랜치**라고 합니다. clone했을 때 생기는 **master** 브랜치는 자동적으로 **origin/master** 브랜치를 추적하게 됩니다. 

**Note**<br><br>로컬 저장소의 리모트 브랜치를 직접 수정할 수는 없지만 `git fetch`를 통해 업데이트 할 수 있습니다.
{: .notice--warning}

### 정리

**리모트 브랜치**
- 리모트 저장소에 있는 브랜치. 
- `git fetch` 명령어를 통해 로컬로 가져올 수 있음.
- 로컬에서는 해당 커밋에 대한 포인터의 역할만 할뿐, 수정할 수 없음
- 예시: origin/master

**트래킹 브랜치**
- 로컬로 가져온 리모트 브랜치를 추적(tracking)하는 브랜치
- 이를 이용하여 변경사항을 리모트 브랜치에 반영
- 예시: clone했을 경우, **origin/master**를 추적하고 있는 **master** 브랜치 

**Upstream 브랜치**
- 트래킹 브랜치가 추적하고 있는 브랜치
- 예시: clone했을 경우, **master** 브랜치의 upstream 브랜치는 **origin/master**

