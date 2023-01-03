---

title: Git 기본 개념과 환경설정에 관하여
author: 김도현
date: 2022-07-16
categories: [Git]
tags: [Git, GitConfig]
math: true
mermaid: true

---

# Git 이란 무엇인가?

> Git은 VCS 프로그램 중 하나이다.

### VCS 란?

> Version Control System 의 약자로, 버전관리를 위한 프로그램을 말한다.

---

# Git 설치 및 환경설정

## 설치

#### 1-1. [Git 공식 홈페이지](https://git-scm.com/) 에서 다운로드 한다.
#### 1-2. 이미 Git 을 설치 했다면 다음 명령어로 업데이트 시도를 해본다.
#### 1-2-1. winget install --id Git.Git -e --source winget
#### 2. 설치과정에서 Git Bash 를 설치하자. (기본값으로 셋팅되어 있음)

#### 3. 설치 확인 -> git --version 명령어를 실행한다.

## 환경설정

#### 1-1. git config --global user.name "원하는 이름"
#### 1-2. git config --global user.email "원하는 이메일"

#### 2. 기본 브랜치 명 변경 -> git config --global init.defaultBranch main
#### 3. 원하는 프로젝트 경로에, git init 으로 git 감시 시작
