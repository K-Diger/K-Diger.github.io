---

title: VPC, Auto Scaling, ELB, Route 53, Bastion Host, IAM, Cloud Watch
author: Diger
date: 2022-12-11
categories: [Cloud, AWS]
tags: [Cloud, AWS]
layout: post
toc: true
math: true
mermaid: true

---

# 1. VPC

VPC를 사용하여 사용자가 정의한 가상 네트워크를 생성한 후 EC2 인스턴스와 같은 AWS 리소스를 그 안에 배치할 수 있다.

즉, 사용자의 워크로드를 외부와 격리되는 네트워크에 구성하여 관리하는 것이 가능하다.

## 1.1. VPC 구성 과정

1. VPC 이름 및 IPv4 CIDR 지정
2. Internet과 통신이 가능하도록 Internet Gateway 생성 및 VPC 연결
3. Subnet 생성 (Seoul 리전의 경우, 가용영역 각각 2a, 2c에 생성한다.)
4. Routing Table 구성 -> 외부의 모든 네트워크와 통신이 가능한 Internet Gateway 경로를 등록한다.
5. 3. 에서 생성한 Public Subnet 리소스들은 Internet과 통신이 가능하도록 RoutingTable을 Public Subnet에 연결한다.
6. Web 서버(EC2 인스턴스)와 DB 서버(RDS)가 사용할 보안 그룹 생성

---

# 2. EC2 Auto Scaling

EC2 Auto Scaling을 사용하면 관리자가 정의한 최소/최대 용량 사이에 구성된 정책에 따라 인스턴스가 자동으로 추가/삭제 된다.

이를 통해 EC2 인스턴스를 **탄력적**으로 운영할 수 있다.

## 2.1. AutoScaling 구성 과정

1. EC2 인스턴스 AMI 생성
2. Launch Template 생성
3. Auto Scaling Group 생성

---

# 3. Elastic Load Balancing (ELB)

Elastic Load Balancing은 온프레미스 환경의 Load Balancer(L4/L7 Switch) 역할을 수행한다.

하나의 가용 영역에 있는 EC2 인스턴스, 컨테이너, IP 주소와 같은 대상에 수신 트래픽을 자동으로 분산한다.

ELB는 대상의 상태를 모니터링하여(Health Check) 정상적으로 동작하는 대상에게만 트래픽을 전송한다.

또한 특별한 설정 없이 ELB 자체적으로 수신 트래픽의 변화에 따라 로드 밸런서의 용량을 자동으로 조정한다.

ELB의 종류는 다음과 같다.

1. Application Load Balancer
2. Network Load Balancer
3. Gateway Load Balancer
4. Classic Load Balancer

## 3.1. Load Balancer 구성 과정

1. AutoScaling 그룹을 사용하여 퍼블릭 서브넷에 다수의 인스턴스 가동
2. Target Group 구성 (요청 트래픽을 전달할 대상들을 지정한다.)
3. AutoScalingGroup(ASG)로 생성된 다수의 인스턴스를 Target Group으로 구성하고, 추후에 생성할 ELB가 해당 Target Group으로 트래픽을 전송하도록 한다.
4. ELB 4가지 유형 중 Application Load Balancer를 생성하고 보안그룹을 구성하여 계층적인 보안 구조를 만든다.
5. Load Balancer의 Listener 를 HTTP 로 구성하여, TCP 80으로 수신한 트래픽을 앞서 생성한 TargetGroup으로 전송하도록 한다. (ASG에 의해 생성된 인스턴스가 자동으로 추가되도록 해야한다.)
6. 타겟 그룹에 해당하는 인스턴스 중 하나를 중지시키면, Unhealthy 상태로 진입하여, 로드밸런서가 해당 인스턴스로 트래픽을 분산하지 않는 결과를 확인한다.
7. Load Baancer를 생성했으면, 기존 EC2 인스턴스를 Private Subnet 으로 이동하여 외부로 부터 존재를 감춘다.
8. ASG에 의해 생성된 인스턴스가 자동으로 Target Group에 구성되도록 하려면, 시작 템플릿에서 기존에 만든 로드밸런서에 연결하는 설정을 해줘야 한다.

---

# 4. Route 53

Private Subnet에 생성된 웹 서비스에 접근하기 위해서는 ALB의 복잡한 도메인 이름을 사용해야한다. 이를 개선할 수 있는 서비스가 Route 53이다.

Route 53은 높은 가용성과, 확정성이 뛰어난 웹서비스 이므로, 사용자에게 빠른 응답이 가능하고 높은 SLA를제공하는 것이 가능하다.

Route 53은

- 도메인 이름 등록
- DNS 라우팅
- 리소스 상태 확인
-
이라는 세 가지 주요 기능을 제공한다.

## 4.1. Route 53 구성 과정

1. 우선 무료 호스팅 사이트에 접속하여 도메인을 부여받는다.
2. Route53 콘솔에서 호스팅 영역을 구성한다. (부여 받은 도메인을 입력한다.)
3. 도메인 등록 사이트에서 네임 서버 주소를 등록한다.
4. Route53 콘솔에서 A 레코드 생성으로 테스트를 한다.
5. ALB에 대한 별칭 레코드를 생성한다. (www)
6. www.부여받은도메인 으로 접속하면 로드밸런서에 접속이 가능한 것을 알 수 있게 된다.

---

# 5. Bastion Host / NAT Gateway/Instance

EC2 인스턴스와 DB 인스턴스는 Private Subnet에 생성되었기 때문에 인터넷과 통신이 불가능하여 관리자 역시 접근이 불가능한 상황이다.

이를 해결하기 위해 Public Subnet에 Bastion Host를 생성하여 관리자는 Bastion Host를 거쳐 Private Subent에 접근할 수 있도록 한다.


---

Bastion Host를 구성하여 외부 관리자가 Private Subnet Resource에 관리 접근이 가능하도록 구성했지만

Private Subnet EC2 Instance 에서 외부 인터넷으로 형하는 요청 메세지는 전송될 수 없는 환경이다.

이때 NAT Gateway를 사용하여 해결할 수 있다.

NAT Gateway는 AWS 관리형 리소스 이기 때문에 이중화, 용량, 대역폭 관리 등을 신경쓰지 않아도 되지만

동작시간과 처리 데이터 양에 따라 과금이 부여된다.

## 5.1. Bastion Host 구성 과정

1. Bastion Host를 보호하기 위한 보안그룹을 생성한다. (조직 관리자의 IP 주소에서만 접근하도록 구성하는 것이 안전하다.)
2. 외부 관리자가 접근 가능하도록 퍼블릭 서브넷에 Bastion Host를 생성한다.
3. 이 설정을 바탕으로 Bastion Host(EC2 인스턴스) 를 생성한다.
4. 기존 EC2 인스턴스와 RDS 인스턴스를 Bastion Host 만 접근 가능하도록 보안 그룹을 수정한다.

## 5.2. NAT Instance 구성 과정

1. NAT Instance에 사용될 보안 그룹을 생성한다. (Private Subnet에서 인터넷으로 전송될 트래픽만 허용하는 것이 좋다.)
2. 고가용성을 위해 가용 영역(a, c) 마다 NAT Instance를 별도로 생성하여 가용영역 이중화를 구성한다.
3. 또한 NAT Instance의 AMI 는 커뮤니티 AMI를 선택한 후 NAT로 검색하여 사용한다. (인스턴스에 NAT가 구성되어 있는 이미지이다.)
4. 가용영역 a에 Public Subnet1을 선택하여 인스턴스를 생성한다.
5. 가용영역 c에 Public Subnet2을 선택하여 인스턴스를 생성한다.
6. 각 생성된 NAT Instance의 소스/대상 확인 기능이 비활성화 인지 확인한다.
7. 그 후 Private Subnet 1,2 가 사용할 라우팅 테이블을 별도로 생성하고, 각 Private Subnet에서 외부 네트워크 대역으로 전송되는 트래픽을 NAT Instace 1,2 로 향하도록 추가한다.
8. Bastion Host에 접속하여 Private Subnet에 존재하는 EC2 인스턴스에 접속한 후, ping 8.8.8.8 명령어를 입력하여 통신가능 여부를 확인한다.

---

# 6.IAM

IAM를 사용하면 AWS 서비스 및 리소스에 접근할 수 있는 보안 주체를 지정하고, 세분화된 권한을 중앙에서 관리할 수 있다.

일상적인 작업에 AWS 계정인 Root User를 사용하지 않을 것을 권고하고 있다. 이를 위해서 IAM 사용자가 등장했고 이에 대한 맞춤 권한을 부여할 수 있다.

IAM USER 생성 후 별도의 권한을 부여하지 않으면 계정 내 서비스 및 리소스에 대한 어떤 접근도 허용되지 않는다.

권한 정책은 JSON으로 작성된다.

조직 내 다수의 IAM User가 존재하고 동일 정책을 부여해야하는 경우 IAM GROUP을 지정할 수 있다.

## 6.1. IAM Role

IAM User와 비슷하게, 특정 작업에 대해 가능/불가능을 결정하는 정책이 포함된 자격증명이다.

이는 임시 자격 증명으로 계정의 리소스에 액세스하려는 사용자 또는 서비스 요청에 사용된다.

사용예시로는 EC2 인스턴스가 S3 버킷을 대상으로 작업하기 위해 S3 작업을 수행할 때

영구적으로 접근 권한을 부여하는 것이 아닌, 시간이 제한되어있는 임시 자격증명을 사용하여 접근할 수 있도록 한다.

---

# 7. CloudTrail

보안과 운영을 위한 서비스로, 사용자 역할 AWS 관리 콘솔, CLI SDK 에서 수행하는 API 호출 내역이 ClouTrail에 기록된다.

이벤트 기록을 통해 지난 90일간의 활동을 확인/검색/다운로드 할 수 있다.

90일이 지난 이벤트 기록을 저장하려면, 추적을 만들어야하는데, 추적은 특정 이벤트를 기록하고 지정한 S3 버킷에 로그 파일을 전달함으로써 로그파일에는 JOSN 형식의 로그 항목이 들어가 있는형태이다.

---

# 8. CloudWatch

모니터링 및 운영 데이터를 로그/지표/이벤트 형태로 수집한다. AWS 리소스/애플리케이션/서비스 운영 상태를 파악하고, 문제해결 + 자동화 작업 + 리소스 사용률 최적화를 수행할 수 있다.

지표는 시스템 성능에 대한 데이터 값을 나타내고. EC2인스턴스, EBS볼륨, RDS 인스턴스에 대한 무료 지표를 제공한다.

EC2 인스턴스의 CPU 사용량/네트워크 전송량/ASG가 관리하는 인스턴스 수 등을 확인할 수 있고

데이터는 최대 15개월 동안 보관된다.

모니터링에는 기본 모니터링과 세부 모니터링 2가지 방식이 있다.

세부 모니터링은 일부 서비스에만 적용되고 비용이 발생된다.

---

## 8.1. CloudWatch Agent

EC2 인스턴스의 운영체제 내부 정보(Memory, Disk)는 수집되지 않기 때문에 이러한 정보를 수집하기 위해선 CloudWatch Agent를 설치하여 수집할 수 있다.

이를 활용하면 시스템 내부 Log 정보 또한 수집하는 것이 가능하다.

CloudWatch Agent를 사용하여 "사용자 지정 지표"를 수집하도록 하려면 IAM Role을 통해 CloudWatch 접근 권한을 EC2 인스턴스에 부여하고, CloudWatch Agent를 설치해야한다.

또한 EC2 인스턴스가 인터넷과 통신이 가능해야한다. 만약 인스턴스가 Private Subnet일 경우 NAT Gateway를 거쳐야한다.

---

## 8.2. CloudWatch Logs

EC2 인스턴스에서 CloudWatch Agnet를 통해 시스템 내부 로그를 수집했으면 CloudWatch Logs로 전송할 시스템 로그를 지정해야한다.

CloudWatch 콘솔에서, 로그 그룹을 생성하고 Agent 구성 파일에서 CloudWatch Logs로 전송할 시스템 로그를 지정하면 된다.

---

## 8.3. CloudWatch Alarms

지표를 감시하여 관리자에게 알림을 보낼 수 있다.

EC2 인스턴스의 경우는 CPU 사용량/디스크IO 등을 모니터링하여 부하가 증가하면 추가 인스턴스를 시작하거나, 중지시킬 수 있다.

ASG의 경우도 CloudWatch Alarms 를 사용하여 ASG 인스턴스들의 평균 CPU 사용량에 따라 ScaleUp/Out을 한다. (자동으로 CloudWatch Alarms가 구성되어 있는 것이다.)

알림은 이메일, 핸드폰으로도 받을 수 있다.

---

## 8.4. CloudWatch Events

CloudWatch Events는 AWS 리소스의 변경 이벤트를 실시간으로 전달한다.

주요 리소스 변경 이벤트 발생 시 해당 이슈에 대한 처리 및 대응을 자동화 할 수 있게한다.

리소스의 변화를 실시간으로 인식하여 간단한 규칙을 통해 이벤트를 분류하고, 처리할 수 있는 대상으로 전달하여 변화에 따르게 대응 가능한 체계를 만든다.

CloudWatch Events -> EventBridge로 대체되었다.

