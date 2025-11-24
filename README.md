## 서버 포트폴리오 LEMP 스택 심화편

### 실습 목적: 시스템 엔지니어가 기존 서버를 인수인계 받은 후 관리 하는 작업을 시뮬레이션 합니다.

### 목차

1. [서버 인수인계 (현재 상태 확인)](01_Server_handover/README.md)
2. 서버 보안 강화 
3. 로그 분석, 장애상황재현, 원인 분석 
4. 성능테스트+병목분석 
5. 자동화 스크립트 고도화

### 실습환경

- 서버 호스트 명: ubuntu-server-01
- 가상화 소프트웨어: VMware Workstation Pro(개인용 라이선스)
- 서버 OS: Ubuntu Server 24.04.3 LTS build 26200.7019
- Kernel : Linux 6.8.0-87-generic
- vCPU/RAM: 8core vCPU, 6GB RAM
- 네트워크 구성<br>
  NAT, 내부 IP(예: 192.168.159.128)<br>
  기본 게이트웨이: 192.168.159.2<br>
  서비스 포트: 22(SSH), 80(Nginx HTTP), 3306(Maria DB)
