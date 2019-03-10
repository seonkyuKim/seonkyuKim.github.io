---
title: "12: 브랜치 합치기 2(git rebase)"
categories: git-tutorial
tags:
- git
- tutorial
author_profile: true
---

저번 시간에 갈라진 두 개의 브랜치를 병합해주는 merge에 대해 배웠습니다. 이번에는 두 브랜치를 병합해주는 또 다른 방법인 `git rebase`를 배우도록 하겠습니다.

다음과 같은 히스토리의 커밋 내역이 있다고 가정합시다:

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-03-05-git-rebase/01.png){: .align-center}

다음 명령어로 rebase를 할 경우 다음과 같이 커밋 히스토리가 하나로 합쳐지게 됩니다:

```
$ git checkout iss15
$ git rebase master
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/2019-03-05-git-rebase/02.png){: .align-center}

원리는 두 브랜치가 나뉘기 전인 공통 커밋으로 이동하고 나서 그 커밋부터 지금 **iss15** 브랜치가 가리키는 커밋까지 diff(차이점)을 차례로 만들어 어딘가에 임시로 저장해 놓습니다. 이후 **master** 브랜치에 **iss15** 브랜치의 커밋들을 차례로 만들고 아까 저장해 놓았던 변경사항을 차례대로 적용합니다. 

