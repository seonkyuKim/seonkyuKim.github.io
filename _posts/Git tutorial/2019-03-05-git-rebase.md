---
title: "12: 브랜치 합치기 2(git rebase)"
categories: git-tutorial
tags:
- git tutorial
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

원리는 두 브랜치가 나뉘기 전인 공통 커밋으로 이동하고 나서 그 커밋부터 지금 **iss15** 브랜치가 가리키는 커밋까지 diff(차이점)을 차례로 만들어 어딘가에 임시로 저장해 놓습니다. 이후 **master** 브랜치에 변경사항들을 차례로 적용하면서 **iss15** 브랜치의 커밋들을 만듭니다. `git rebase`의 장점은 커밋 히스토리가 깔끔해진다는 점입니다.

**Note**<br><br>`git merge`와는 반대로, **iss15** 브랜치로 이동한 후 **master**브랜치로 rebase한다는 점에 주의하십시오.
{: .notice--warning}

## 주의사항

절대로 리모트 저장소에 push한 코드에 대해서는 `git rebase`를 사용하지 마십시오. 이는 커밋 히스토리를 조작하기 때문에 엄청난 혼란을 야기합니다. 자세한 내용에 대해서는 '[3.6 Git 브랜치 - Rebase하기](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-Rebase-%ED%95%98%EA%B8%B0#_merge_rebase_work)'의 예시를 참고하시기 바랍니다.

## Merge vs Rebase

커밋 히스토리의 중요한 역할 중 하나는 **작업한 내용의 기록**입니다. 이런 관점에서 `git rebase`로 작업 내역을 바꾸는 것은 좋지 못하다고 볼 수 있습니다. 하지만, 수많은 merge 커밋 히스토리를 그대로 냅두는 것은 괜찮을까요?

히스토리를 **프로젝트가 어떻게 진행되었나**에 대한 관점으로 볼 수도 있습니다. 이런 관점으로는 다른 사람들에게 보여주기 좋도록 `git rebase`가 좋은 무기가 될 수 있습니다.

결론은, 각자의 상황에 따라 `git merge`와 `git rebase`를 적절히 사용할 수 있어야 합니다. 단, 어딘가에 Push로 보낸 커밋에 대해서는 절대 Rebase하지 마십시오.
