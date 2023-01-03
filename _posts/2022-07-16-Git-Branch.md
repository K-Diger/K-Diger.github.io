---

title: Branch 전략, 명령어 정리
author: 김도현
date: 2022-07-16
categories: [Git]
tags: [Git, Branch]
math: true
mermaid: true

---

# Git-Flow

| 브랜치     | 역할 |
|---------|-------------------|
| main    | 제품으로 출시될 수 있는 브랜치 |
| develop | 다음 출시 버전을 개발하는 브랜치 |

<br>

|보조브랜치| 역할                       |
|---|--------------------------|
| feature | 기능을 개발하는 브랜치             |
| release | 이번 출시 버전을 준비하는 브랜치(QA)   |
| hotfix | 출시 버전에서 발생한 버그를 수정하는 브랜치 |

![Git-Flow](https://user-images.githubusercontent.com/60564431/179346591-d0edee5e-1bff-4600-aee0-330590bdffde.jpg)

> 위 그림은 브랜치 전략을 그려본 내용이다.
>
> 보조 브랜치는, 병합 후 삭제되어야 한다.

<br>

> 다음과 같은 브랜치 전략을 수행하기 위해서, 브랜치를 다루는 명령어를 알아보자.

---

# Branch Create


## 1. 로컬 저장소 브랜치 생성하기 (Local Folder)
> git branch 브랜치이름

## 2. 원격 저장소 브랜치 생성하기 (GitHub)
> git push origin ##1.에서생성한 브랜치이름

---

# Branch Read

## 1. 브랜치 확인하기
> git branch

## 2. 브랜치 변경하기 (해당 브랜치이름으로 브랜치 변경)
> git checkout 브랜치이름

---

# Branch Delete

## 1. 로컬 저장소 브랜치 제거하기 (Local Folder)
> git branch -d 브랜치이름

## 2. 원격 저장소 브랜치 제거하기 (GitHub)
> git push origin :브랜치이름

---

# Branch Update

> 브랜치 이름 변경 방법은, 변경하고싶은 이름의 브랜치를 생성한 후, 기존 브랜치를 제거하는 것이다.


## 1. 로컬 저장소 브랜치 이름 변경 (Local Folder)
> git branch -m 기존브랜치이름 변경브랜치이름


## 2. 원격 저장소 브랜치 생성 (GitHub)
> git push origin 변경브랜치이름

## 3. 원격 저장소 기존 브랜치 제거 (GitHub)
> git push origin :기존브랜치이름 변경브랜치이름
