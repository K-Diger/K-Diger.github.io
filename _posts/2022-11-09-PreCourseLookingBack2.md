---

title: 우아한테크코스 5기 프리코스 2주차 회고

date: 2022-11-04
categories: [프리코스]
tags: [프리코스]
layout: post
toc: true
math: true
mermaid: true

---

# 시작

이번 주차부터는 저번주에 느낀점을 바탕으로 요구사항에 대한 문서를 꼼꼼하게 읽어보기로 했다.

Fork&Clone 을 시작으로 기능목록을 작성하면서 어떻게 해야지 깔끔하게 코드를 분리할 수 있을지에 생각하는게 먼저였다.

---

# 위기 1.

이번 주 과제시작부터 문제가 발생했다. 요구사항에서 제공하는 Console.readline() 메서드가 실행되지 않던 것이였다.

에러 메세지는 다음과 같았다.

![](../images/wooahPrecourse/2주차 골칫덩이.png)

이러한 에러는 For, While 반복문 내에서 Console.readline을 사용하면 동일하게 발생했다.

에러를 잡기위해 구글링을 하면서 리플렉션 문제다, readline 메서드가 입력의 끝을 알 수 없어서 발생하는 문제다. 라는 원인의 후보지들을 알게되었지만...

> 둘 다 아니였다. 직접 디버깅을 하며 찾아봤을 때도 이러한 원인이라고 생각이 되었지만 결정적으로 이 문제가 아니였다.
>
> 현재 사용하고 있는 Java 버전의 문제였다. 어떤 이유인진 정확하게 파악하진 못했지만
>
> 18 버전을 쓰고 있던 내 환경이 우아한테크코스에서 제공하는 11 버전 기반의 메서드를 처리하지 못했던 것이다.

---

# 미쳐 인지하지 못했던 위기 1.

단위테스트를 작성하지 않았다.

요구사항 문서에 있던 내용이었지만 이 내용은 제출한 후 다음 주 미션을 Fork 하는 과정에서 알게 되었다.

만약 이게 실제 서비스 과정에서의 요구사항이었고 이를 배포할때 까지 인지하지 못했다면 나는 어떻게 되었을까?

> 회사에서 짤리지 않았을까?


# 2주차 종료

요구사항을 정확하게 분석하고, 내가 다뤘던 모든 기능에 대해서 꼼꼼하게 검토해야한다.

이 행동이 선택사항이나 시간이 남을때 하는 것이 아닌, 필수적인 사항이라는 것을 다시 한 번 되새기고 다음 주차 미션은 정말 완벽하게 제출하고 싶다.

기능 구현이나, 그 외의 요구사항에 대한 내용을 반영하는 작업 자체는 너무나도 재미있었다.

프리코스만을 거치면서도 Java에 대한 친밀도가 올라갔다고 생각하며, 피드백과 요구사항을 바탕으로 어떻게 코드를 작성하는 것이

나의 개발 습관에 도움이 되는지 깨달은 주차였다.
