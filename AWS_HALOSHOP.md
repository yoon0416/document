
#  HALO_SHOP 3차 프로젝트 배포 가이드 (EC2 + React + Spring Boot + MySQL + Nginx)
> 작성자 안윤기

## 📌 서버 정보
- ec2 : t2.medium
- 퍼블릭 IP: `54.180.80.252`
- EC2 접속:
  `ssh -i "haloshop.pem" ubuntu@ec2-54-180-80-252.ap-northeast-2.compute.amazonaws.com`
- **반드시 코드 잘 보시고 본인 ec2 ip로 바꾸시길 바랍니다.**
- **또한 위 ec2 ip와 pem 키는 작성자의 것이며 교육용으로 만든 예시기 때문에 해당 코드 그대로 할 시 존재하지 않아 오류가 발생할 것 입니다.**
- 해당 예시는 kakaopay와 이미지경로가 완벽히 수정된 버전이 아니기 때문에 해당파트 사람이 직접 고치거나 s3 사용하시길 바랍니다 ㅎ

#### 반드시 보안그룹에서 3000포트와 8080포트 허용
#### 개인적으로 vscode에서 3분활#  HALO_SHOP 3차 프로젝트 배포 가이드 (EC2 + React + Spring Boot + MySQL + Nginx)

## 📌 서버 정보
- ec2 : t2.medium
- 퍼블릭 IP: `54.180.80.252`
- EC2 접속:
  `ssh -i "haloshop.pem" ubuntu@ec2-54-180-80-252.ap-northeast-2.compute.amazonaws.com`
- **반드시 코드 잘 보시고 본인 ec2 ip로 바꾸시길 바랍니다.**
- **또한 위 ec2 ip와 pem 키는 작성자의 것이며 교육용으로 만든 예시기 때문에 해당 코드 그대로 할 시 존재하지 않아 오류가 발생할 것 입니다.**
- 해당 예시는 kakaopay와 이미지경로가 완벽히 수정된 버전이 아니기 때문에 해당파트 사람이 직접 고치거나 s3 사용하시길 바랍니다 ㅎ

#### 반드시 보안그룹에서 3000포트와 8080포트 허용
#### 개인적으로 vscode에서 cmd 열 때 3개 분할해서 열고 하나는 front 하나는 back 하나는 window(jar파일 업로드용도)로 작업하는 거 추천드립니다!
---

## 1. IP 치환 작업

### 1-1. 프론트와 백엔드의 URL을 EC2 IP 기준으로 통일 
> 뒤에 나올 간편한 코드가 있기 때문에 jar파일인 backend만 일단 고쳐주시길 바랍니다!!!! 또한 vscode에서 전체검색(ctrl + shift + f) 사용하여 `localshot:8080` 검색 후 백만 우선적으로 다 바꿔줍니다. 
- React에서 요청하는 백엔드 주소(`localhost:8080`) → `http://54.180.80.252:8080`  
- Spring에서 리턴하거나 이미지 접근 등 프론트 주소(`localhost:3000`) → `http://54.180.80.252`
> 백엔드는 8080붙이지만 프론트는 3000 안붙임!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
<br>

- 예시: 아이템 이미지 업로드 경로 설정  (스프링, 일단 동작을 위해 이것만 해도 다른 대부분 기능 테스트 가능)
> /items/ItemsImageController.java
```java
String uploadDir = "/home/ubuntu/uploads";
```

백엔드 다 하셨다면 메이븐 클린 후 메이븐 인스톨 그리고 pem키가 있는 폴더에 jar파일 넣어주세요!

---

## 2. pem 키 보안 오류 해결 (Windows PowerShell)

> pem 키 파일이 타 사용자에게 오픈되어 있을 경우 EC2 접속이 차단됨. 아래 명령어로 보안 제한 적용

```
icacls "D:\haloshopaws\haloshop.pem" /inheritance:r
icacls "D:\haloshopaws\haloshop.pem" /remove "NT AUTHORITY\Authenticated Users"
icacls "D:\haloshopaws\haloshop.pem" /grant:r "$($env:USERNAME):R"
icacls "D:\haloshopaws\haloshop.pem" /remove "BUILTIN\Users"
```

---

## 3. EC2 계정 비밀번호 설정

> root/ubuntu 계정의 비밀번호를 설정해 서버 접근 및 sudo 작업 시 활용

```
sudo passwd root
sudo passwd ubuntu
```

---

## 4. Java 11 설치 및 환경변수 설정

> Spring Boot 앱 실행을 위한 Java 11 설치 및 시스템 환경변수 등록

```
sudo apt-get update
sudo apt-get install openjdk-11-jdk -y
sudo vi /etc/profile
```

`/etc/profile`에 추가:
```
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=$CLASSPATH:$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar
```

적용 및 확인:
```
source /etc/profile
echo $JAVA_HOME
clear
```

---

## 5. MySQL 8 설치 및 사용자/비밀번호 설정

> DB 연동을 위한 MySQL 설치 및 root 비밀번호 설정, haloshop DB 생성

```
sudo apt-get install mysql-server-8.0 -y
sudo mysql -uroot
```

MySQL 내부에서 실행:
```
use mysql;
UPDATE user SET plugin='caching_sha2_password' WHERE user='root';
flush privileges;
SET PASSWORD FOR 'root'@'localhost' = '1234';
exit
```

DB 생성:
```
sudo mysql -uroot -p
create database haloshop;
```

---

## 6. 리눅스 및 MySQL 시간대 설정

> 시스템 및 DB 시간대를 서울(KST)로 변경

```
sudo timedatectl set-timezone Asia/Seoul
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

추가 내용:
```
default-time-zone = '+09:00'
```

재시작:
```
sudo systemctl restart mysql
```

---

## 7. 서버로 파일 전송 및 Git 클론

> jar 파일 전송 및 GitHub에서 프로젝트 클론 (Windows PowerShell)

```
scp -i "D:\haloshopaws\haloshop.pem" "D:\haloshopaws\haloshop.jar" ubuntu@54.180.80.252:/home/ubuntu/
git clone https://github.com/joyulbi/HALO_SHOP.git
```

---

## 8. React 프로젝트 package.json 설정

> React 앱 실행/배포 스크립트 정의

```json
"scripts": {
  "dev": "next dev",
  "build": "next build",
  "start": "next start"
}
```

---

## 9. Node 및 npm 설치

> React 앱 실행을 위한 Node.js 및 패키지 매니저 설치

```
sudo apt-get update
sudo apt-get install -y build-essential
sudo apt-get install curl
sudo apt-get install nodejs -y
sudo apt-get install npm

node -v
npm -v
```

> 의존성 강제 설치 (버전 충돌 방지용)
```
sudo npm install --legacy-peer-deps
sudo npm install react-is
```

---

## 10. React 앱 빌드 및 실행

> 실제 운영용 파일로 빌드 후 실행

```
sudo npm run build
```

빌드 오류 시 권한 수정:
```
sudo chmod -R 775 .next
sudo chmod -R 775 .
sudo chown -R ubuntu:ubuntu .
```

---

## 11. PM2 (Node 프로세스 관리자) 설치

> React 앱을 데몬으로 실행하거나 자동 재시작하려면 사용

```
sudo npm install pm2 --legacy-peer-deps
```

---

## 12. Nginx 리버스 프록시 설정

> 포트 80으로 들어온 요청을 React(3000) 및 Spring(8080)으로 분기

포트 확인 및 종료:
```
sudo su
sudo lsof -i tcp:80
sudo kill -9 <PID>
```

설치 및 기존 설정 제거:
```
sudo apt-get install -y nginx
sudo rm /etc/nginx/sites-available/default
sudo rm /etc/nginx/sites-enabled/default
```

프록시 설정 파일 생성:
```
cd /etc/nginx/sites-available
sudo vi proxy.conf
```

proxy.conf 내용:
```
server {
    listen 80;

    location / {
        proxy_pass http://localhost:3000;
        ...
    }

    location ~ ^/(api|movies|member) {
        proxy_pass http://localhost:8080;
        ...
    }
}
```

링크 및 재시작:
```
sudo ln -s /etc/nginx/sites-available/proxy.conf /etc/nginx/sites-enabled/proxy.conf
sudo nginx -t
sudo service nginx restart
```

---

## 13. localhost 경로 일괄 검색 및 치환

> 코드 내 하드코딩된 URL을 EC2 IP 기준으로 변경

검색: 이건 수동으로 직접 바꾸세요
```
grep -rl "localhost:3000" .
```

치환: 이건 그냥 이렇게 해도 됨
```
find . -type f \( -name "*.js" -o -name "*.jsx" -o -name "*.ts" -o -name "*.tsx" \) \
  -not -path "*/node_modules/*" -not -path "*/.next/*" -not -path "*/.git/*" \
  -exec sed -i 's|http://localhost:8080|http://54.180.80.252:8080|g' {} +
```

### 코드 돌려도 중간에 빠진게 있을 수 있으니 만약 서버 안되면 하나하나 .js 확인하고 바꿔주세요 ㅎ

---

## 14. front서버 실행 및 Spring Boot 실행 및 로그 확인

빌드 & 실행: (이건 맨 마지막에 하세요 뒤에 다 하고)
```
npm run build && npm start
```
<br>

백엔드 서버 실행 및 로그 실시간 확인 + 서버 실행할 때 시간대 서울로 기준점
```
nohup java -Duser.timezone=Asia/Seoul -jar haloshop.jar > haloshop.log 2>&1 &
tail -f haloshop.log
```

---
# 여기서부턴 필요하다면 사용

## 15. Spring Boot 강제 종료

```
kill -9 $(ps aux | grep 'haloshop.jar' | grep -v grep | awk '{print $2}')
```

---

## 16. 포트 사용 확인

```
lsof -i :8080
```
