---
title: "Git의 다양한 머지 전략 비교 - 우리 팀은 어떤 전략을 도입해야 할까?"
layout: post
date: 2021-07-11 01:03
tags:
- git
categories: 
- Tech
- Engineering
language: ko
toc: true
widgets:
   - type: toc
     position: left
   - type: categories
     position: left
   - type: recent_posts
     position: left
---
Git은 한 브랜치에서 작업한 내용을 Main 브랜치에 병합(Merge)할 수 있는 다양한 방법들을 제공한다. 이러한 방법들을 **Merge 전략**이라고 부르는데, 다양한 방법들 중에서도 이번 글에서는 가장 많이 사용되는 방법인 1) Merge Commit, 2) Squash and Merge, 3) Rebase and Merge에 대해 소개하려고 한다.

<!--more-->

<img src="/assets/images/git-merge-strategy-base.png?style=centerme" alt="현재 브랜치와 commit 상태" width=53%>


위의 그림과 같은 상태의 commit이 생성되었다고 가정하자. `feat/multiply`라는 브랜치가 있고, `feat/sum`이라는 브랜치가 있다. 각 commit 내의 숫자는 commit의 global 순서를 나타낸다. 

## Merge Commit

<img src="/assets/images/git-merge.png?style=centerme" alt="(좌) Merge Commit의 개념도 (우) Merge Commit 이후 Github에서의 commit log" width=92%>

브랜치의 commit log와 merge log가 동시에 기록된다. Commit log는 commit을 행한 순서대로 기록되고, merge log는 merge가 된 순서대로 기록된다. 동시에 기록되기 때문에 commit log가 verbose해지며 commit log의 순서가 merge 순서와 다르기 때문에 history 관리 및 이해가 어렵다. 

## Squash and Merge

<img src="/assets/images/git-squash-merge.png?style=centerme" alt="(좌) Squash and Merge의 개념도 (우) Squash and Merge 이후 Github에서의 commit log" width=82%>

Merge된 순서대로 master/main 브랜치에 기록된다. 그리고 작업 완료된 브랜치의 commit은 새로운 commit 으로 모두 squash되며, 새로운 commit의 제목은 PR 제목이 되고, 합쳐진 commit의 제목은 새로운 commit의 상세 내용이 된다. 

이러한 특징 때문에 master/main 브랜치의 히스토리 관리가 쉬우나, atomic commit level로 rollback 하는 것은 불가능하다. 

## Rebase and Merge

<img src="/assets/images/git-rebase-merge.png?style=centerme" alt="(좌) Rebase and Merge 이후 main 브랜치의 변화 (우) Rebase and Merge 이후 Github에서의 commit log" width=60%>

Commit 순서가 아닌 merge 순서대로 기록된다. 그래서 하나의 PR에 담긴 commit message가 다른 PR의 commit message와 섞이지 않는다. 그리고 rebase 덕분에 merge된 이후의 로그를 보았을 때 하나의 브랜치에서 연속적으로 작업한 것과 같은 로그를 확인할 수 있다. 이 때문에 얼마든지 항상 원하는 수준으로 rollback 이 가능하다. 

하지만 잘 적용하기 위해서는 commit을 생성할 때부터 올바른 commit 단위로 분리해야 하며, commit message 또한 설명력을 가지고 있어야 한다. 그리고 다른 PR이 먼저 merge되는 경우, rebase 작업이 필요할 수 있고 이 때 발생할 수 있는 conflict를 잘 해결할 수 있어야 한다.

## Summary

Merge Strategy | Pros | Cons 
-- | -- | --
**Merge Commit** | 아직 찾지 못함 | 불필요한 commit message가 생기고 merge 순서와 commit 순서가 별도로 기록되어 history 관리가 어려움
**Squash and Merge** | Commit 단위 별로 꼼꼼하게 관리하지 않아도 PR title 만 제대로 관리하면 history가 깔끔하게 정리됨 | Atomic level의 rollback이 어려움 
**Rebase and Merge** | Atomic level의 rollback이 용이하며 commit 단위의 history 기록이 됨 | Commit을 잘 다루지 못하는 경우, rebase에 익숙하지 않은 경우 어려움이 발생

앞서 소개한 전략들의 장/단점을 정리하면 위와 같다. 개인적으로는 아직 **Merge Commit**의 장점을 발견하지 못했다. 결론적으로 팀원 전체가 git을 다루는데에 굉장히 익숙해서 commit 단위, commit message, rebase 등에 어려움이 없는 경우, 혹은 atomic level의 rollback이 필요한 개발 상황에서 **Rebase and Merge**가 선호된다. 하지만 atomic level 까지의 rollback은 필요하지 않고 팀이 이제 막 git으로 버전관리하는 법을 배우기 시작했다면 **Squash and Merge**가 좋을 것이다. 
