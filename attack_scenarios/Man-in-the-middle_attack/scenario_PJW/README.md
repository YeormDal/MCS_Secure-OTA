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

## Step 1 : OTA 서버 실습 환경 만들기
### 1.1 실습 목표
- OTA 업데이트 서버를 Flask로 직접 만들고, 차량이 업데이트를 받는 것처럼 위장된 서버를 운영해보자.
- Flask : Python 기반으로 만든 가벼운 웹 서버 프레임워크 
  - 복잡한 설정 없이, 코드 몇 줄로 웹 서버를 만들 수 있으며, 빠르게 개발하고, 가볍게 실험하는 데 딱 맞는 도구
  - Apache처럼 서버 역할을 할 수 있는데, 훨씬 가벼움
 
### 1.2 디렉토리 구성
- 실습에 사용할 폴더 구조를 만들어야 한다.
![image](https://github.com/user-attachments/assets/6d0846f4-5a69-423f-959f-111032243110)

## Step 2 : Flask OTA 서버 만들기
### 2.1 Flask 설치하기
![image](https://github.com/user-attachments/assets/71bd937f-5dfb-4292-b6ec-3940a15f2b89)

### 2.2 Flask OTA 서버 코드 만들기
- nano로 코드 파일 생성
![image](https://github.com/user-attachments/assets/a625287e-6dba-4f29-a1fc-7bbb95a018a2)
- app.py 파일에 코드 작성
![image](https://github.com/user-attachments/assets/4012a366-06a7-43bd-aea4-3b1f564a850d)
  - Ctrl X (나가기) → Y (내용 저장) → Enter (저장 확정)
  - nano로 파일 생성/수정 시, 위 순서로 저장 가능 (앞으로도 계속 사용) 

## Step 3 : HTTPS 인증서 직점 만들어서 Flask에 연결하기
### 3.1 실습 목표
- 지금까지 만든 Flask 서버는 기본적으로 HTTP로 통신!
  - HTTP는 데이터가 암호화되지 않아서, 중간에 누군가 통신 내용을 몰래 엿보거나 조작할 수 있다.
- HTTP → HTTPS (보안 HTTP)로 서버를 변경
  - HTTPS를 쓰면, 서버와 클라이언트 간의 데이터가 암호화되어 안전하게 주고 받을 수 있음!
- 웹 서버가 HTTPS로 통신하려면 인증서 (SSL/TLS 인증서)가 필요
  - 인증서가 없으면 브라우저나 클라이언트가 "이 사이트는 안전하지 않습니다"라는 경고를 띄움!
- 따라서, OpenSSL을 사용해서 자가 서명 인증서(self-signed certificate)를 직접 만들어야 한다.

- 실습 목표 : 이 인증서를 Flask 서버에 연결해서 HTTPS 통신을 직접 구현하는 것

### 3.2 cert 디렉토리로 이동
- 여기에 인증서 파일을 저장할 예정
  
![image](https://github.com/user-attachments/assets/68b838b7-0227-4cb7-86cd-115774630f63)

### 3.3 OpenSSL로 인증서 만들기
![image](https://github.com/user-attachments/assets/e2f102d8-e48f-419c-86bb-664a48ced8bf)
- 1년동안 사용할 수 있는, 비밀번호 없이 바로 쓸 수 있는 자가 서명 HTTPS 인증서(ota.crt)와 키(ota.key)를 만든다!

### 3.4 중간에 이런 입력창이 나옴 
![image](https://github.com/user-attachments/assets/b734b8a7-75f7-41ae-87a9-2ac0b7cb7420)
- Common Name(CN) : ＂인증서 안에 적힌 서버의 주민등록증 이름”이다.
  - 접속하려는 서버 주소와 다르면, 브라우저나 클라이언트가 믿지 않기에, ”ota.realserver.com” 으로 입력!
- 나머지 요소들은 인증서에 들어가는 부가정보이므로 개별적으로 자유롭게 작성

### 3.5 인증서가 잘 생성됐는지 확인
![image](https://github.com/user-attachments/assets/bb1ec7cd-043d-440e-b8ed-82134f493512)
- ota.crt / ota.key 두 파일이 보이면 성공!
- 추가로 tree 명령어로 전체 구조 확인 가능! 
![image](https://github.com/user-attachments/assets/1411d2a1-189c-4c2a-bedf-0cc22b68ce10)

### 3.6 cat 명령어로 파일 살펴보기
![image](https://github.com/user-attachments/assets/c7433d68-d648-49c4-81af-798abad33a8d)
- 인증서 안에 긴 공개키, 발급정보 등의 정보 볼 수 있음
- 이걸 통해 HTTPS 통신을 암호화

### 마무리
- 이제 FLask 서버는 ssl_context=(‘certs/ota.crt’, ‘certs/ota.key’) 옵션을 읽어서, HTTPS로 안전하게 통신 가능!
  - 인증서를 직접 만들었다. ✔️
  - certs/ 폴더에 안전하게 저장했다. ✔️
  - Flask 서버에 연결할 준비를 끝냈다. ✔️

## Step 4 : 악성 OTA 패키지 생성 및 서명
### 4.1 실습 목표
- 차량은 OTA 업데이트를 받을 때, 파일이 변조되지 않았는지 해시/서명 검증을 한다.
- 우리는 악성 OTA 파일 (firmware.bin)을 만들고, 공격자 키로 서명
- 진짜 서버를 속일 준비를 할 예정

### 4.2 updates 디렉토리로 이동
- OTA 업데이트 파일(firmware.bin)과 관련 파일(hash, signature)을 이 폴더에 모음!
![image](https://github.com/user-attachments/assets/9cd82e41-1f96-4a33-84ed-96d0d5cc1477)

### 4.3 악성 OTA 파일 생성
- firmware.bin 파일 생성 → 내용은 간단하지만, 우리는 이걸 OTA 업데이트로 속일 예정! 
![image](https://github.com/user-attachments/assets/8237abdc-487b-4f8e-be87-c2c9175dd0f9)
- 내용 확인
![image](https://github.com/user-attachments/assets/ffda7de1-0589-4793-ab84-0c65fbd8dac3)

### 4.4 firmware 해시값 만들기 (SHA-256)
![image](https://github.com/user-attachments/assets/99459906-08b3-4fc3-9567-1fdb1dac8ecc)
- 해시(hash)는 파일 전송 중에 변조되지 않았는지 확인하는 값
- firmware.hash 파일 안에 해시값이 기록됨
- 내용 확인
![image](https://github.com/user-attachments/assets/b63a0c9c-481e-4689-b21b-63ebd834c96a)

### 4.5 공격자 키 생성(개인 키)
![image](https://github.com/user-attachments/assets/af1df00e-2ac9-410c-a4ff-f535ebb0138c)
- 공격자가 자기 사인을 할 때 사용할 비밀 개인 키 생성
- 이 키로 OTA 파일을 “내가 서명했어요~” 확인 표시 
- 확인
![image](https://github.com/user-attachments/assets/5a894cd6-8f34-4465-9bf0-283a0724784b)

### 4.6 공격자 공개 키 생성
![image](https://github.com/user-attachments/assets/6f4abfdf-b07a-4f4e-a386-366334906449)
- 차량(클라이언트)은 공개키만 가지고 “서명이 진짜인지” 확인함!
- 내용 확인
![image](https://github.com/user-attachments/assets/3b7a6690-8648-4424-9e73-2e68101af7e4)

### 4.7 firmware 파일에 디지털 서명하기
![image](https://github.com/user-attachments/assets/cfb5961e-bc09-44a0-bb48-f3cb97e1dd2e)
- firmware.bin 파일을 공격자 키로 서명해서 firmware.sig 파일을 만든다.
- 확인
![image](https://github.com/user-attachments/assets/a25858b2-d48e-4d34-bb56-e8369cce4ee8)

### 4.8 현재까지 파일 확인
![image](https://github.com/user-attachments/assets/42c8779a-4f23-4a2e-8779-bb04d81e9a38)

### 마무리 
- OTA 서버에서 제공할 파일 3개 (firmware.bin / firmware.hash / firmware.sig) 준비 완료!
- 이제 클라이언트는
  - firmware.bin을 다운로드하고
  - hash 파일과 비교하고
  - signature를 검증해서
  - “아 이건 정상 OTA”구나 라고 착각하게 만들 수 있음!

  - 악성 업데이트 준비 완료 ✔️
  - 공격자 서명까지 완료 ✔️

- 다음 단계는 Flask 서버에 파일 제공하고, 클라이언트 시뮬레이터를 만들어서 실제로 다운받고 설치하는 것!

## Step 5 : Flask 서버 띄우고 클라이언트 시뮬레이터 만들기
### 5.1 Flask OTA 서버 띄우기
- 1. server 디렉토리로 이동
  - 아까 app.py 만들어둔 곳으로 이동!
  ![image](https://github.com/user-attachments/assets/433c89a0-39b3-471a-9b6b-8f89beb35f35)
- 2. Flask 서버 실행
  ![image](https://github.com/user-attachments/assets/e633631f-c92d-44fd-b3f7-f072427005eb)
  - sudo 쓰는 이유? : 443 포트(HTTPS 포트)는 리눅스에서 관리자 권한이 필요
  - 이제 내 컴퓨터(공격자 노트북)가 HTTPS OTA 서버가 된 것이다!
- 3. 서버 상태 확인
  - 새 터미널 열고 (서버는 계속 켜둔 채)
  ![image](https://github.com/user-attachments/assets/3ad1db8a-dc04-446b-90a3-27034f971318)
  - Malicious firmware simulation file → 정상 출력 

### 5.2 클라이언트 시뮬레이터 만들기
- 1. ota.cleint 디렉토리로 이동
  ![image](https://github.com/user-attachments/assets/39147ad4-f370-4dd5-9e28-e701eea28fad)
- 2. client.simulator.py 파일 만들기
  ![image](https://github.com/user-attachments/assets/2e49c3f3-93f0-4a49-85e9-8e9d33b402f3)
  ![image](https://github.com/user-attachments/assets/6ca5b6dd-2336-43db-be42-29a664deb29c)
- 3. 공개 키 복사
  ![image](https://github.com/user-attachments/assets/8cda030d-7c98-4bdf-bb6f-2fe5b76df05e)
- 4. 시뮬레이터 실행
  ![image](https://github.com/user-attachments/assets/0afbf6a3-3d29-4655-a01c-66a26fdf196b)

### 마무리
- Flask로 가짜 OTA를 세웠고
- 악성 업데이트를 만들었고
- 클라이언트를 속여서 설치하게 했다.

