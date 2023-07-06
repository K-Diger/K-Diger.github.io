---

title: JSCODE - 자바 스터디 5회차 (GitHub) + 심화 미션(Git Flow + rebase)

date: 2023-02-16
categories: [Git]
tags: [Git, Branch]
layout: post
toc: true
math: true
mermaid: true

---

# 공부해 볼 내용!

> 디미터 법칙

- Git Flow

- GitHub Flow

- GitLab-Flow

- Trunk-Based

- Rebase

각 전략마다 특징이 뭔지 파악하자. (23/02/16)

# Git Flow

## Git flow 란?

Git 을 사용하면서 얻고자 하는 이점은 "분산 버전 관리"일 것이다.

Git을 활용한다면 한 프로젝트 레포지토리에 대해서 브랜치를 나누고, 각 브랜치 별로 별도의 작업을 수행하여 독립적으로 버전을 관리하는 것이 가능해진다.

이때, "브랜치를 나누고" 라는 문장에서 브랜치를 나누는 것에 대한 전략이 있다. 그리고 이것을 Git Flow 라고 한다.

## Git flow 의 종류?

- feature

기능의 구현을 담당, feature/구현 중인 기능 와 같이 네이밍 하는게 일반적이다.

feature 는 develop 브랜치의 하위 브랜치이며, 기능 제작이 완료되면 feature 브랜치는 삭제된다.

- develop

feature에서 기능이 개발될 때마다 합쳐지는 브랜치이다.

어느정도 완성된 기능이 많아진다면, 배포를 위한 브랜치인 release로 병합된다.

- release

release 브랜치는 develop 브랜치에서 생성되고, 버그 수정 내용을 develop 브랜치에도 반영한다.

어느정도 완성된 기능들이 모인 브랜치이다.

실 배포를 위한 브랜치라기 보단 배포 시 테스트를 위한 브랜치이다. 완전하게 준비되었다면 master(main) 브랜치로 병합한다.

- hotfix

master(main) 브랜치에서 생성되고 develop 브랜치, release 브랜치와 master(main) 브랜치에 수행한 내용을 반영한다.

배포된 소스에서 버그가 발생되면 이를 처리하기 위한 브랜치이다.

hotfix-N 으로 네이밍을 하는 것이 일반적이다.

- master(main)

최종 배포되는 브랜치이다.

## 각 Branch의 책임 요약

| 브랜치          | 역할 |
|--------------|-------------------|
| master(main) | 제품으로 출시될 수 있는 브랜치 |
| develop      | 다음 출시 버전을 개발하는 브랜치 |

> 알쓸신잡 : master는 Black lives matter 운동으로 인해 main으로 변경되었다.

|보조브랜치| 역할                       |
|---|--------------------------|
| feature | 기능을 개발하는 브랜치             |
| release | 이번 출시 버전을 준비하는 브랜치(QA)   |
| hotfix | 출시 버전에서 발생한 버그를 수정하는 브랜치 |

![Git-Flow](https://user-images.githubusercontent.com/60564431/179346591-d0edee5e-1bff-4600-aee0-330590bdffde.jpg)

위 그림은 브랜치 전략을 그려본 내용이다.

보조 브랜치는, 병합 후 삭제되어야 한다.

![Git-Flow](https://user-images.githubusercontent.com/43775108/125800526-2ea36d8e-6262-4ba5-9ef0-af7845131d85.png)

위 그림은 가장 이상적인 Git Flow 흐름을 나타내는 그림이다.

다음과 같은 브랜치 전략을 수행하기 위해서, 브랜치를 다루는 명령어를 알아보자.

---

## Branch Create

### 1. 로컬 저장소 브랜치 생성하기 (Local Folder)

```shell
git branch 브랜치이름
```

### 2. 원격 저장소 브랜치 생성하기 (GitHub)

```shell
git push origin ##1.에서생성한 브랜치이름
```

## Branch Read

### 1. 브랜치 확인하기

```shell
git branch
```

### 2. 브랜치 변경하기 (해당 브랜치이름으로 브랜치 변경)

```shell
git checkout 브랜치이름
```

---

## Branch Delete

### 1. 로컬 저장소 브랜치 제거하기 (Local Folder)

```shell
git branch -d 브랜치이름
```

### 2. 원격 저장소 브랜치 제거하기 (GitHub)
```shell
git push origin :브랜치이름
```

---

# Branch Update

> 브랜치 이름 변경 방법은, 변경 하고싶은 이름의 브랜치를 생성한 후, 기존 브랜치를 제거하는 것이다.


## 1. 로컬 저장소 브랜치 이름 변경 (Local Folder)

```shell
git branch -m 기존브랜치이름 변경브랜치이름
```

## 2. 원격 저장소 브랜치 생성 (GitHub)

```shell
git push origin 변경브랜치이름
```

## 3. 원격 저장소 기존 브랜치 제거 (GitHub)

```shell
git push origin :기존브랜치이름 변경브랜치이름
```

---

# GitHub Flow

[참고하기 좋은 블로그](https://sihyung92.oopy.io/architecture/gitflow-vs-githubflow#c609762c-92c6-4e20-a997-d839056941be)

아마 우리가 보통 Git을 처음 쓰고 프로젝트를 진행하면서 가장 많이 사용해왔던 패턴이 아닐까 하는 초간단 전략이다.

부연 설명할 것이 없이 아래 그림으로 모든 설명이 끝난다...

![GitHub-Flow](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcSQzqk%2FbtqUTnAMFqj%2FdliROOijRpnhkVncaTEloK%2Fimg.png)

위 그림처럼 main브랜치는 항상 배포가 가능한 상태에 있으며 모든 브랜치는 main에서 시작한다는 특징과

이렇게 뻗어나간 브랜치들은 **반드시** PR후에 Merge가 되는 방식을 제안하는것이 GitHub Flow이다.

정말 단순하고 쉽게 사용할 수 있는 점이 큰 강점으로 생각되지만 규모가 큰 프로젝트에서는 과연 살아남을 수 있는 전략인지 궁금하긴하다!

---

# GitLab Flow

[참고하기 좋은 블로그](https://sihyung92.oopy.io/architecture/gitflow-vs-githubflow#c609762c-92c6-4e20-a997-d839056941be)

GitHub Flow는 너무 간단해서 배포, 환경 구성, 릴리즈, 통합 에 대한 이슈가 많다고 한다.

그래서 이를 보완하고자 GitLab에서 제안한 전략이다.

![GitLab-Flow](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcKg4cN%2FbtrsC9Iih0f%2FXl7RoK2KfrzWVkWttjfKNk%2Fimg.png)

위 그림이 GitLab Flow를 잘 나타내는 그림이다.

master 에서 기능 개발을 수행하고 (필요 시 feature 브랜치도 만들어서 기능 개발을 한다.)

완성된 기능을 가진 내용을 pre-production 에 반영하고

그 이후에 production 에 반영하는 형태이다.

즉, 앞서 살펴본 GitHub Flow에서 배포를 위한 브랜치를 따로 추가하는 것으로 생각하면 된다.

---

# Trunk Based Development

GitHub Flow와 비슷하게 보이는 방식이다.

trunck 혹은 master(main) 브랜치에서 모든 작업을 수행하는 방식인데 여기서 반드시 지켜야하는 규악이 있다는 점이 가장 큰 차이점이다.

## Trunk Based Development Rule 1. 페어 프로그래밍

페어 프로그래밍은 한 명은 개발, 한 명은 코드 리뷰를 실시간으로 수행하는 방식인데

PR 요청이 추가로 필요하지 않고 즉각적인 피드백을 적용할 수 있다는 장점으로

Trunk Based Development의 꼭 지켜야하는 규악 중 한 가지로 채택되었다.

## Trunk Based Development Rule 2. 빌드의 신뢰성

현재 반영된 코드가 항상 빌드가 가능한가? 를 고려해야한다.

이를 위해선 자동테스트와 효과적인 테스트 전략이 요구된다.

## Trunk Based Development Rule 3. 작업이 완료되지 않은 내용이 있는가?

main 브랜치에는 미처 작업이 완료되지 않은 내용이 있을 수 있다.

그렇기 때문에 feature 브랜치를 사용하여 작업이 완료되지 않은 부분은 숨겨야한다.

이 때 완료되지 않은 내용이 단순한 작업이라면 branch by abstraction

다소 복잡한 내용이라면 feature 브랜치를 사용하면 된다.

## Trunk Based Development Rule 4. 소규모 개발

릴리즈를 더 자주 할 수 있도록 소규모 단위로 쪼개서 개발한다.

소규모 단위의 적절한 양은 며칠이 아닌, 몇 시간안에 완료될 수 있는 양을 말한다.

## Trunk Based Development Rule 5. 빠른 빌드

빌드/테스트 작업은 수 분 이내로 완료되어야한다. 이 때 작업시간이 길어진다면 아키텍처적인 개선을 고려해봐야한다.


## Trunk Based Development 의 장점

- 빠른 피드백

- 코드 컨벤션

- CI

- 병합 충돌 방지 -> 대규모 리팩터링 가능

---

# Rebase

rebase는 같은 뿌리를 가진 2개 이상의 Branch에서

Branch A의 Base를 Branch B의 최신 커밋으로 base를 옮기는 것이다.

이렇게 되면, Branch B의 그동안의 커밋 내용을 모두 깔끔하게 가져가 더 보기좋은 커밋 히스토리를 만들 수 있지만

한 커밋마다 매번 충돌을 해결해줘야하는 상황도 발생하긴한다.

rebase를 사용하는 명령어와 어떨 때 쓰이면 좋을지는 조금 더 알아본 후 작성해보자 (2023/02/18)
