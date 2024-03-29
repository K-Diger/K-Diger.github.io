---

title: 우아한테크코스 5기 프리코스 1주차 회고

date: 2022-11-04
categories: [프리코스]
tags: [프리코스]
layout: post
toc: true
math: true
mermaid: true

---

# 시작 전

혹시 모른다는 생각에 문제를 공유하진 않았습니다.

문제에 대한 정보는, 아래 문제 풀이 레포지토리를 참고해주세요

---

# 1주차 시작

[온보딩 레포지토리](https://github.com/K-Diger/java-onboarding)

지원할때부터 떨렸다. 본 코스가 아님에도 본 코스로 가기 위한 과정인지라 잘 해내고 싶은 마음이 컸다.

본 코스에 가고자 하는 계기는 많은 이유가 있지만 다음 3가지로 축약한다.

1. 나와 관심사가 같은 인원들과 약 1년가까이 되는 시간을 몰입할 수 있다.
2. 1번 과정에서 현업자분들의 피드백을 흡수할 수 있다.
3. 특정 개념에 대해서 무언가에 쫓기지 않고 깊숙하게 몰입하여 학습해보고 싶다.

---

# 위기 1.

#시작 에서 말했듯이, 잘 해내고 싶은 마음에 요구사항을 꼼꼼히 읽지 않았다. 오직 기능 구현에만 초점을 맞췄다.

이런 유형을 현재 듣고있는 학교수업 교수님께서는 "코더" 라고 한다던데 "코더"가 되고 말았다.

**기능 단위 커밋**, **기능 목록 작성** 등을 수행하지 않고 오직 테스트코드에만 의존하여 요구사항에 만족하는 출력 결과가 나오는지만 확인했다.

> 2022/11/04 클린아키텍처 스터디에서 진행한후로 깨달았다. 기능단위 커밋과 기능 목록 작성을 왜 해야하는가?

클린 아키텍처와 DDD 의 빌드업 과정이었던 것이다.

현재 읽고있는 클린 아키텍처에서 도메인(어떠한 요구사항을 충족 시킬 수 있는 로직 단위) 단위로 개발을 한 후에 Controller 와 같은 Web Adapter를 생성하라는 내용이 있었다.

이 내용을 읽고 번뜩였다. 그래서 기능 목록을 작성하라 했구나, 그래서 기능단위(도메인 단위) 로 커밋을 해야하는구나,...

물론 이 내용을 1주차 프리코스에 깨달은 것은 아니고, 현재 글을 작성하고 있는 약 10분 전에 알게 되었다.

---

# 위기 2.

구현된 코드가 너무 길다. 나는 교내 프로젝트에 참여하면서 개발에 본격적으로 입문한 유형이다.

Java/Spring 기반으로 프로젝트를 진행했고, 서버를 맡았지만 개발 속도가 더딘 이유로 다른 파트에 피해를 주는 상황까지 벌어져

Java 에 대한 학습도 얕게했고 오직 Spring에서의 DB 접근 기술에만 신경썼었다.

이런 영향이 있었는지, 순수 Java 코드로만 기능을 작성하는데에 불편한 점이 많았다.

또한 이전 기수들의 후기대로 한 메서드가 하나의 기능을 갖도록 작성하다보니 메서드가 여러개로 분리되고

그에 따라서 코드의 양 또한 길어졌다.

> 물론 가장 큰 문제점은 .... 내가 Java에서 제공하는 코드를 활용하지 못한 탓도 클 것이다.

---

# 위기 3.

Pull Request 를 보내는 과정에서, 나는 Fork 한 레포지토리에 대해 Clone을 한게아니라 Code 를 압축파일로 받아 사용했었다.

그러다보니 실제 온보딩 레포지토리에 대한 Git 정보를 내 로컬 git 이 인식할 수 없어서 Pull Request를 보낼 수 없는 상황이 벌어졌다.

제출이 몇 시간 남지 않았는데,, 너무 아깝고 초조했었다.

결국에는 remote 를 다시 연결하고 rebase 하는 것으로 해결했지만 이 문제를 예방할 방법 또한 이미 온보딩 진행방식 문서에 적혀있었다.

제발 문서를 꼼꼼히 읽자 기능 구현만 하는 "코더"가 되지 말자 라고 다시 한 번 느꼈다.

---

# 1주차 종료

문제는 대체적으로 풀만했다. 뭔가 풀릴것같은데...? 라는 생각이 든 문제가 많았고 또 그 생각이 들때면 귀신같이 1~2시간 내로 해결되었었다.

또한 슬랙에서 테스트케이스를 공유해주시는 많은 분들에게 감사함을 느끼고 내 로직을 점검했었다.

문자열을 다룰 때 charAt() 만 사용했던 나에게 substring()을 알려주신 분도 감사하다.

이렇게 원래 알고 있던 방법도 막상 데드라인이 정해지면 마음이 조급해져서 생각이 안나는 듯하다.

마음이 급하면 오히려 역효과가나니 앞으로는 멘탈 관리에도 신경 써야겠다고 느꼈다.
