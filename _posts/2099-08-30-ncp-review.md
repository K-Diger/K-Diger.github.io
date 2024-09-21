---

title: NCP 적용 후기
date: 2024-08-30
categories: [NCP]
tags: [NCP]
layout: post
toc: true
math: true
mermaid: true

---

# Q. 프로젝트 소개

[개발 중인 서비스](https://piikii.co.kr)

모임 관리 서비스로 모임에 참여할 인원들과 함께 모임 계획을 구성할 수 있는 서비스이다.

함께 모임 후보지를 모아오는 `방`이라는 개념이 있고 해당 방에 장소를 추가하여 일행들과 어떤 장소들이 있는지 상호 공유할 수 있고

지정한 시간이 되면 추가해놓은 장소들을 투표하여 투표에서 선정된 장소들을 모아 하나의 모임 코스를 만들어주는 서비스.

---

# Q. Ncloud 활용 서비스

- Server
- VPC
- Object Storage
- Container Registry

---

# Q. Ncloud 서비스를 어떻게 적용 하였나요?

![](https://github.com/mash-up-kr/piikii_Spring/raw/develop/docs/architecture/overall.png)

API, 모니터링 서버를 띄우는데 사용했으며 정적 파일을 보관하기 위한 Object Storage를 사용했다.

그리고 서버를 이미징한 도커 이미지를 보관하기 위한 Container Registry를 사용하여 이미지를 관리하고 있다.

---

# Q. Ncloud 사용 중 특히 만족했던 점과, 아쉬웠던 점은 무엇인가요?

한국어 문서화가 된 것이 마음에 들었다. 하지만 비용이 꽤나 비싸서 크레딧이 금방 소진되는 점이 아쉬웠다.

Object Storage를 사용하기 위한 SDK 문서가 꽤나 오래전 버전의 SDK 라이브러를 사용하게 하는 것으로 보였다. 최신 버전에도 대응할 수 있도록 되었으면 좋겠다.

---

# Q. Green Developers 프로그램 참여 소감 말씀 부탁 드립니다. (50자 이상)

앞으로도 동일한 기회가 주어진다면 더 다양한 서비스들을 써보고 싶습니다. 한국 친화적인 서비스라 편의성에 만족하고 갑니다.

---

# Q. 마지막 한 말씀 부탁 드립니다.

NCP서비스가 번성하길 바랍니다. 더 좋은 서비스 제공 해주세요!
