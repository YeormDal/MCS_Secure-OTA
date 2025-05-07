
# 위장 OTA 서버를 통한 악성코드 유포 

## 1. 시나리오 개요 및 기초 준비물

### 1.1 시나리오 개요 
- 공격자는 실제 OTA 업데이트 서버를 위장하여 차량의 DNS 요청을 공격자의 서버로 유도하고 악성 OTA 업데이트를 설치하게 한다.

### 1.2 기초 준비물
- VMware : OTA 서버 및 클라이언트 가상 머신 구동
- dnsmasq : DNS 요청을 공격자의 서버(Flask OTA 서버)로 리디렉션
- Bettercap : DNS Spoofing 및 MITM 공격 도구
- Flask(Python) : 위장 OTA 서버 운영
- OpenSSL : OTA 패키지 서명 및 위조 인증서 생성

## 2. 실습 단계
### 1단계 : OTA 서버 및 DNS 환경 구축
- Kali Linux 가상 머신 환경에 Flask 기반 OTA 서버 구축
- dnsmasq 설정을 통해 DNS Spoofing 환경 구성

### 2단계 : DNS Spoofing 설정
- Bettercap을 이용하여 차량의 DNS 요청을 공격자의 OTA 서버 IP로 리디렉션 설정
- ota.realserver.com → 공격자 IP로 강제 변경

### 3단계 : OTA 서버 위장 및 HTTPS 설정
- Flask 서버를 HTTPS(SSL 인증서)로 설정하여 가짜 OTA 서버 구축
- OpenSSL을 사용하여 자가 서명 HTTPS 인증서 생성 및 적용

### 4단계 : 악성 OTA 패키지 생성 및 서명
- OTA 업데이트 패키지(firmware.bin) 생성
- OpenSSL을 이용하여 OTA 패키지 해시값 생성 및 공격자 키로 디지털 서명 생성

### 5단계 : OTA 업데이트 요청 및 악성 패키지 전송
- 클라이언트 시뮬레이터가 OTA 서버에 접속하여 firmware.bin, signature, hash 다운로드
- 다운로드한 파일의 해시 및 서명을 검증하여 OTA 설치 여부 판단
- 악성 OTA가 정상적으로 설치되는지 검증
