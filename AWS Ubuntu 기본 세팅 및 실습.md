# 🚀 AWS Ubuntu 기본 세팅 가이드 (SSGFC 프로젝트 기준)
제발제발제발제발제발 한줄씩 복붙하세요
> - 본 설정은 **팀 프로젝트 1: SSGFC** 기준입니다.  
> - **ec2 t2.meddium**,**ebs 30gb**, **Ubuntu 22.04**, **Java 11**, **MySQL 8**, **Spring Boot 내장 톰캣 사용**
> - 프로젝트는  **Java 11**, **MySQL 8**, **Spring Boot 2.7.14**
> - **파일질라(FireZilla)** 기반 파일 업로드/다운로드  
> - 로그, JAR, SQL, TMP는 모두 `~/ssgfc` 폴더 기준
> - 참고로 ssgfc main껄로 업로드 시도 시 막힘 (따로 자바 설정한거 있음 귀찮아서 git push + 병합 안함) 완벽하게 만든 후 깃 병합할 시 메모하겠습니다
> - 해당 문서는 학원 쌤 강의내용 + 공식문서 + 유튜브 + chatgpt + 구글링 등을 활용하여 만든 문서입니다.
> - 정보 더블체크 하시고 만약 실습 때 작동이 정상적으로 안된다면 문의해주시거나 ai 활용해주시길 바랍니다.
> - 실습 이미지도 넣고 싶으나 시간상 나중에 넣겠습니다

---

## ⚠️ 주의사항

- 인스톨 시 `-y` 옵션으로 `y` 입력 스킵 가능
- Spring Boot 내장 톰캣 사용 시 별도 톰캣 설치 안 해도 됨
- `vi`에서 insert 모드 하는 경우가 아니라면, 한 줄씩 복붙 권장
- ssgfc.jar 안에 내장톰캣이 있기 때문에 우분투 안에서 톰캣설정 안하고 실행했음.


---

## 🔐 계정 및 기본 설정

```bash
sudo passwd root
sudo passwd ubuntu
```

---

## ☕ Java 11 설치 및 환경 변수 설정

```bash
sudo apt-get update
sudo apt-get install openjdk-11-jdk -y
sudo vi /etc/profile
```

`/etc/profile`에 다음 내용 추가:

```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=$CLASSPATH:$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar
```

```bash
source /etc/profile
echo $JAVA_HOME
clear
```

---

## 🌀 (선택) Tomcat 9 설치 및 방화벽 설정

```bash
sudo apt-get install tomcat9 -y
sudo ufw allow 22/tcp
sudo ufw allow 8080/tcp
sudo ufw enable
sudo ufw status
sudo service tomcat9 start
sudo service tomcat9 stop
systemctl status tomcat9
```

> ※ Spring Boot 내장 톰캣으로 충분히 가능하므로 생략 가능

---

## 🌐 (선택) 네트워크 도구 설치 및 포트 확인

```bash
sudo apt-get install net-tools
netstat -plnt | grep ':8080'
clear
```

---

## 🐬 MySQL 8 설치 및 기본 설정
> - mysql열 때 'sudo'까지 안할 시 권한 부족으로 update, insert 못할 수 있음

```bash
sudo apt-get install mysql-server-8.0 -y
sudo mysql -uroot
```

```sql
-- MySQL 내부
use mysql;
UPDATE user SET plugin='caching_sha2_password' WHERE user='root';
flush privileges;
SET password for 'root'@'localhost'='1234';
exit
```

```bash
sudo mysql -uroot -p
```

> ※ DB 생성 시 `create database dbname;` 입력

---

## 🏃 Spring Boot 실행 (SSGFC 프로젝트 기준)

```bash
cd ~/ssgfc
nohup java -jar ssgfc.jar & jobs
```

또는 로그까지 출력:

```bash
nohup java -jar ssgfc.jar > log.txt 2>&1 &
jobs
tail -f log.txt
```

---

## 🧩 DB 적용 (last.sql import)

```bash
mysql -h localhost -P 3306 -uroot -p1234 ssgfc
mysql -h localhost -P 3306 -uroot -p1234 ssgfc < last.sql
SHOW TABLES;
```

---

## 🖼️ 이미지 추출 및 업로드 경로 설정

```bash
cd ~/ssgfc
unzip ssgfc.jar "BOOT-INF/classes/static/images/*" -d tmp

mkdir -p /home/ubuntu/uploads/{userimage,players,board,logo}
cp -r tmp/BOOT-INF/classes/static/images/userimage/* /home/ubuntu/uploads/userimage/
cp -r tmp/BOOT-INF/classes/static/images/players/* /home/ubuntu/uploads/players/
cp -r tmp/BOOT-INF/classes/static/images/board/* /home/ubuntu/uploads/board/
cp -r tmp/BOOT-INF/classes/static/images/logo/* /home/ubuntu/uploads/logo/
chmod -R 755 /home/ubuntu/uploads/
```

---

## 🌐 WebConfig.java - 정적 리소스 매핑

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/images/userimage/**")
            .addResourceLocations("file:/home/ubuntu/uploads/userimage/");
    registry.addResourceHandler("/images/players/**")
            .addResourceLocations("file:/home/ubuntu/uploads/players/");
    registry.addResourceHandler("/images/board/**")
            .addResourceLocations("file:/home/ubuntu/uploads/board/");
    registry.addResourceHandler("/images/logo/**")
            .addResourceLocations("file:/home/ubuntu/uploads/logo/");
}
```

---

## ✅ 변경된 사항 요약

1. **인스턴스 유형 `t2.medium`으로 업그레이드**
   - 탄력적 IP 사용 안 함
   - 과금 관련된 정리는 따로 md 만들 예정

2. **우분투 내부에 이미지 파일 직접 업로드**
   - `/home/ubuntu/uploads` 경로에 이미지 저장
   - WebConfig에서 해당 경로 정적 매핑
3. 수정할 사항
   - 프론트와 백엔드 서버 각각 독립적으로 하여 효과적으로 빠른 속도로 이미지 랜더링 가능하게
   - 이미지 불러오는 로직 등 변경하여 트래픽 최소화 및 성능개선 필요
   - aws 업로드 기준이 아닌 로컬에서 실행했을 때 동작하는지 처음에 구상하고 만든 프로젝트다 보니 aws버전으로 최적화 필요
---
