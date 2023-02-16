---

title: JSCODE - 자바 스터디 5회차 (GitHub) + 심화 미션(Git Flow + rebase)
author: 김도현
date: 2023-02-16
categories: [Git]
tags: [Git, Branch]
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

[Git-Flow](https://user-images.githubusercontent.com/43775108/125800526-2ea36d8e-6262-4ba5-9ef0-af7845131d85.png)

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
