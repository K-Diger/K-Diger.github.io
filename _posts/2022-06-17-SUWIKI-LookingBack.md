---

title: SUWIKI 마무리 전 고민거리
author: 김도현
date: 2022-06-17
categories: [Project, SUWIKI]
tags: [SUWIKI]
math: true
mermaid: true

---

# 고민거리

재직자들의 포트폴리오를 보면

> 연간 ~~~~ 회 API 요청 성능 ~~ 최적화

같은 문구가 있는 것을 보게되었다.

이참에 나도 궁금해졌다.

### 내가 직접 만든 API 서버는 하루에 몇 회씩 요청을 받아서 처리해줄까?

이 궁금증을 해결할 수 있을 것 같다는 생각이 하나 있다.

## 생각 1.

**DB에 테이블을 만든다.**

테이블의 **컬럼**은

**DateTime 타입의 ApproachTime,**

**Long 타입의 Count**

위 두개로만 둔다.

그래서 어떤 API 요청을 하던간에, 그 API 메서드 상단에, 혹은 인터셉터 - 필터로 그 요청이 들어왔을때

그 날짜에 해당하는 DB의 Count에 하나씩 증가시켜서 하루의 총 API 요청량을 알아낸다.

## 생각 2.

아직 없음...
