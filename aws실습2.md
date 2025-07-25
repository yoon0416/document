## 1. EC2 접속
1. 본인 퍼블릭 ip 확인
2. 인스턴스 연결 SSH 클라이언트 클릭
  > 맨 밑에 예: 링크 복사
3. cmd에 링크 복사한 거 붙여넣고 yes입력
  > 보안이 너무 널널하다고 에러뜨면 설정해주면 됨 (chatgpt 도움 ㄱ)

4. 보안설정 때문에 cmd에서 우분투 안들어가질 경우(한줄씩 실행) 이건 예시임
```
icacls "D:\springreact\springreact.pem" /inheritance:r
icacls "D:\springreact\springreact.pem" /remove "NT AUTHORITY\Authenticated Users"
icacls "D:\springreact\springreact.pem" /grant:r "$($env:USERNAME):R"
icacls "D:\springreact\springreact.pem" /remove "BUILTIN\Users"
```

## 2. 파일 올리기 - scp 
1. 다른 cmd창 하나 더 열기 (우분투 들어간 cmd아님.)
   
```bash
scp -i "어떠한 키로" " 어떤 파일을" 어떤 aws 인스턴스에 올리겠다

ex)
scp -i "D:\xxx\s3tt.pem" "D:\xxx\board.jar" ubuntu@x5.xx4.xxx.xxx:/home/ubuntu/
```

---
# 리눅스 설정

##  RAM - SWAPFILE (만약 프리티어로 어거지로 하겠다면)
```linux bash
sudo fallocate -l 8G /swapfile : 8g 빈파일
sudo chmod 600 /swapfile : 소유자 읽고 쓰기 권한
sudo mkswap /swapfile :포맷
sudo swapon /swapfile : 사용가능하게 시스템 활성화
sudo swapon --show : 현재사용되는 스왑영역 표시
sudo free -h : 메모리 상태확인
```

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

## 🏃 Spring Boot 실행
- 귀찮으니 로그는 모르겠고 이걸로 ㄱ
```
nohup java -jar ex1.jar & jobs
```


---
---
---

## s3 예시
---

## 2. spring + security + jpa + s3

##### 1. [EC2-S3]

1. 네비게이션 - 서비스 - S3
 > 버킷만들기 : react-node-d2big , 리전설정 , ACL 비활성화 - 내가 접속한 계정에서만 소유하도록 , 동의 □ 모든 퍼블릭 액세스 차단 풀기
2. [권한]탭
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AddPerm",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::sally03915/*"
        }
    ]
}
```
- 3. 내이름탭 클릭 - 보안자격증명 -  새 액세스키만들기 - 키 다운로드



##### 2. [SPRING] 
0. S3파일설정

```pom.xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-aws</artifactId>
    <version>2.2.1.RELEASE</version>
</dependency>
```

```app
cloud.aws.credentials.access-key=엑세스키
cloud.aws.credentials.secret-key=시크릿키
cloud.aws.region.static=ap-northeast-2
cloud.aws.s3.bucket=버킷명
cloud.aws.stack.auto=false
```

> s3 설정클래스
```bash
import com.amazonaws.auth.AWSStaticCredentialsProvider;
import com.amazonaws.auth.AWSCredentials;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.AmazonS3ClientBuilder;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class S3Config {

    @Value("${cloud.aws.credentials.access-key}")
    private String accessKey;

    @Value("${cloud.aws.credentials.secret-key}")
    private String secretKey;

    @Value("${cloud.aws.region.static}")
    private String region;

    @Bean
    public AmazonS3 amazonS3() {
        AWSCredentials credentials = new BasicAWSCredentials(accessKey, secretKey);
        return AmazonS3ClientBuilder.standard()
            .withRegion(region)
            .withCredentials(new AWSStaticCredentialsProvider(credentials))
            .build();
    }
}
```
> 파일업로드 클래스 설정 / 기존 설정 변경
```bash
package com.thejoa.boot3.util;

import java.io.IOException;
import java.util.UUID;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;

import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.model.ObjectMetadata;

@Component
public class UtilUpload {

    private final AmazonS3 amazonS3;

    @Value("${cloud.aws.s3.bucket}")
    private String bucketName;

    public UtilUpload(AmazonS3 amazonS3) {
        this.amazonS3 = amazonS3;
    }

    public String fileUpload(MultipartFile file) throws IOException {
        // 1. 고유한 파일 이름 생성
        String fileName = UUID.randomUUID().toString() + "_" + file.getOriginalFilename();

        // 2. 메타데이터 설정
        ObjectMetadata metadata = new ObjectMetadata();
        metadata.setContentLength(file.getSize());
        metadata.setContentType(file.getContentType());

        try {
            // 3. S3에 파일 업로드
            amazonS3.putObject(bucketName, fileName, file.getInputStream(), metadata);
        } catch (IOException e) {
            e.printStackTrace(); // 예외 처리
            throw e;
        }

        // 4. 업로드된 파일의 S3 URL 반환
        return amazonS3.getUrl(bucketName, fileName).toString();
    }
}

```

> detail.html (버킷설정)

1. spring boot - pom.xml ( lombok )

2. maven install

##### 3 

---
# sql
- cmd에서
```
scp -i "C:\Users\sallyAn\Desktop\aws_20250615\spring+mybatis+react.pem" "C:\Users\sallyAn\Desktop\aws_20250615\thejoa.sql" ubuntu@13.209.49.199:/home/ubuntu/
```
- 우분투에서
```
mysql -u root -p thejoa  < thejoa.sql
```



# react

##### 14. 노드 설치하기 
```bash
sudo  apt-get  update
sudo  apt-get  install  -y  build-essential
sudo  apt-get  install  curl
sudo  apt-get  install nodejs  -y
sudo  apt-get  install npm

node -v
npm  -v

sudo npm install          --legacy-peer-deps
```

##### 15. build

```bash
sudo npm install react-is
```

```bash
sudo npm run build
```

##### 16. pm2

```bash
sudo npm install pm2 --legacy-peer-deps
```


##### 17. nginx ?

```bash
sudo su
sudo lsof -i tcp:80
(나오는 값이 없어야 함, 나온다면 sudo kill -9 프로세스아이디(PID))
```

```bash
sudo apt-get install -y nginx
```

```bash
sudo rm /etc/nginx/sites-available/default
sudo rm /etc/nginx/sites-enabled/default
```

```bash
cd /etc/nginx/sites-available
sudo vi proxy.conf
```

```bash
server {
    listen 80;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Cookie $http_cookie;
    }

    location ~ ^/(api|movies|member) {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Cookie $http_cookie;
    }
}
```

~ 정규표현식을 사용하겠다는 의미입니다 (^ 같은 패턴이 들어갈 때 필요) <br>
^/() 요청 URL이 /로 시작하고 안의 내용(posts 등)과 일치하는지 확인

```bash
sudo ln -s /etc/nginx/sites-available/proxy.conf /etc/nginx/sites-enabled/proxy.conf
```

```bash
sudo nginx -t

```

```bash
sudo service nginx restart
```

##### 18.

```bash
sudo npm start && sudo npx pm2
npx pm2 logs
npx pm2 kill
sudo npx pm2 list
sudo npx pm2 reload all
sudo npx pm2 stop app

sudo lsof -i tcp:3065
sudo lsof -i tcp:80
```

```bash
sudo chown ubuntu:ubuntu uploads
sudo chmod 755 uploads
```





