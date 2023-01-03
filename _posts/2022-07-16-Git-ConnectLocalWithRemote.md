---

title: Git 로컬 저장소와 원격 저장소 연결
author: 김도현
date: 2022-07-16
categories: [Git]
tags: [Git, Branch]
math: true
mermaid: true

---

## 1. Github에 repository 생성


---

## 2. 원격 저장소(repository)와 연결할 로컬 폴더의 경로에서 CMD 열기


---

## 3. git init

> git 초기화

---

## 4. git remote add origin 레포지토리 주소

> origin은 원격 저장소의 주소를 뜻하는데 이를 설정해주는 작업이다.

---

## 5. git pull origin 브랜치이름

> branch를 가져온다.

---

## 6. git add .

> 로컬폴더의 내용을 추가한다. 만약 특정 파일만 올리고 싶다면, . 대신 파일 명을 넣어주면 된다.

---

## 7. git commit -m '커밋 메세지'

> 커밋한다.

---

## 8. git push -u origin 브랜치이름

> github에 올린다.


---

# git push -u origin main --> 오류날 때


### 1. 현재 로컬 저장소의 branch 를 확인한다.

    git branch

아마 브랜치를 확인하면 높은 확률로 master branch 로 설정이 되어있을 것이다

### 2. 현재 로컬저장소의 branch 이름을 변경한다.

    git branch -M 변경할브랜치이름

### 3. 다시 push 한다.
    git push --set-upstream origin main
