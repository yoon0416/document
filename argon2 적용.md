# argon2
> - https://crates.io/crates/argon2
---

## 1. 암호화 알고리즘 4가지 장단점

| 알고리즘   | 장점                                                                 | 단점                                                                  |
|------------|----------------------------------------------------------------------|-----------------------------------------------------------------------|
| **BCrypt** | - 널리 검증된 안정성<br>- 비용 인자(cost factor) 조절 가능             | - 메모리 사용 적음<br>- GPU 공격에 상대적 취약                         |
| **SCrypt** | - 메모리+CPU 하드닝으로 GPU 공격 방어<br>- Spring Security 내장 지원   | - 메모리와 CPU 사용량 부담 큼<br>- 리소스 제한 환경에 부담 가능         |
| **Argon2** | - 최신 알고리즘<br>- 메모리, CPU, 병렬성 조절 가능<br>- GPU/ASIC 공격에 강함 | - 구현 복잡성<br>- 환경 의존성 존재<br>- 파라미터 미설정 시 보안 취약 가능<br>- 비교적 신생 알고리즘 |
| **PBKDF2** | - 오랜 기간 검증된 표준<br>- Java JDK 기본 지원                       | - 메모리 사용 적음<br>- GPU 병렬 공격에 취약                            |

---

## 2. Argon2 세팅 및 메모리 설명
- 아르곤2는 spring 시큐리티(5이상), java 몇버전인지 기억은 안나는데 11이면 충분함
  - 특정 버전 이하부터는 지원 x
- 주요 파라미터:  
  - 메모리 사용량 (예: 8MB = 8192 KB)  
  - 반복 횟수 (iterations)  
  - 병렬성 (parallelism)

- 비밀번호 해싱 1회당 약 8MB 메모리 사용 (설정값 기준)  
- 동시 요청 예시:  
  - 로그인 3명 + 회원가입 2명 동시 → 약 5 × 8MB = 40MB 메모리 사용 예상  
- 리소스 과부하 방지를 위해 파라미터 조절과 동시 처리 제한 필요

### Argon2 Java 예시 코드 (Spring Security)

```java
import org.springframework.security.crypto.argon2.Argon2PasswordEncoder;

public class PasswordEncoderConfig {
    public Argon2PasswordEncoder argon2PasswordEncoder() {
        int memory = 8 * 1024;       // 8 MB 메모리 사용
        int iterations = 2;          // 반복 횟수 2
        int parallelism = 1;         // 병렬 처리 1
        int hashLength = 32;         // 해시 길이 32 bytes
        int saltLength = 16;         // 솔트 길이 16 bytes

        return new Argon2PasswordEncoder(memory, iterations, parallelism, hashLength, saltLength);
    }
}
```
---

## 3. AWS t2.medium 환경 고려 및 오토스케일링 개념

- **AWS t2.medium 사양**  
  - CPU: 2 vCPU  
  - 메모리: 4GB  
- **동시다발 로그인/회원가입 요청 시 문제점**  
  - CPU와 메모리 자원이 제한되어 있어 과부하 발생 가능  
  - Argon2 같은 무거운 해시 연산이 많으면 리소스 소모 심함  
- **대응책**  
  - **오토스케일링(Auto Scaling) 활용**  
    - 인스턴스 수를 자동으로 늘리고 줄여서 부하 대응  
  - 요청 제한(Rate Limiting), 지연 처리(Throttling), 큐잉 등 다층 방어 병행 필요

---

### 오토스케일링(Auto Scaling) 개념

- **자동으로 EC2 인스턴스 수를 조절해주는 AWS 서비스**  
- **특징**  
  - 최소, 최대, 원하는 인스턴스 수를 설정 가능  
  - CPU 사용률, 네트워크 트래픽 등 모니터링 지표에 따라 인스턴스 증감  
  - 트래픽 급증 시 인스턴스를 자동 생성해 부하 분산  
  - 트래픽 감소 시 불필요한 인스턴스 종료해 비용 절감  
- **장점**  
  - 트래픽 변화에 유연한 대응 가능  
  - 서비스 안정성 및 가용성 향상  
  - 비용 효율적인 리소스 운영 가능

---

> 요약:  
> t2.medium 같이 제한된 리소스 환경에서는 오토스케일링을 통해 서버 대수를 자동으로 조절하여,  
> 급격한 트래픽 변화에도 과부하 없이 안정적으로 서비스를 운영할 수 있음.


---
## 4. AWS 로드 밸런서 개념 및 유형별 특징

### 로드 밸런서란?

- 여러 서버(인스턴스)에 들어오는 네트워크 트래픽을 분산시켜  
  단일 서버 과부하 방지 및 서비스 가용성 향상에 도움을 주는 장치(서비스)
- AWS에서는 Elastic Load Balancing(ELB) 서비스를 제공하며,  
  주로 세 가지 유형이 있음

---


---
## 4. AWS 로드 밸런서 개념 및 유형별 특징

### 로드 밸런서란?

- 여러 서버(인스턴스)에 들어오는 네트워크 트래픽을 분산시켜  
  단일 서버 과부하 방지 및 서비스 가용성 향상에 도움을 주는 장치(서비스)
- AWS에서는 Elastic Load Balancing(ELB) 서비스를 제공하며,  
  주로 세 가지 유형이 있음

---

### AWS 로드 밸런서 3가지 유형

| 유형                   | 주요 기능 및 특징                                             | 장점 및 적합 환경                                   | 단점                                    |
|----------------------|----------------------------------------------------------|----------------------------------------------|---------------------------------------|
| **Application Load Balancer (ALB)** | - 7계층(애플리케이션 계층) 로드 밸런싱<br>- HTTP/HTTPS 트래픽 최적화<br>- URL 경로, 헤더 등 세밀한 라우팅 가능 | - 웹 애플리케이션, REST API 서비스에 최적<br>- SSL 종료 및 인증 지원<br>- 스프링 기반 HTTP 서비스에 적합 | - TCP/UDP는 지원하지 않음<br>- NLB 대비 약간 높은 레이턴시 |
| **Network Load Balancer (NLB)**      | - 4계층(TCP/UDP) 로드 밸런싱<br>- 초고성능, 초저지연 처리 가능<br>- 고정 IP 지원                   | - 고성능 TCP/UDP 서비스, 낮은 지연 시간 요구 환경<br>- 게임 서버, 실시간 통신 등에 적합        | - HTTP 레벨 세밀 라우팅 불가<br>- SSL 종료 불가          |
| **Gateway Load Balancer (GWLB)**     | - 네트워크 트래픽 가상화 및 서드파티 네트워크 어플라이언스(방화벽, IDS 등) 통합 관리에 특화           | - 네트워크 보안 솔루션 배포에 최적<br>- 복잡한 네트워크 정책과 보안 환경에 적합            | - 일반 웹 애플리케이션에는 부적합<br>- 설정 복잡           |

---

### Spring Java 웹 프로젝트에 가장 적합한 로드 밸런서

- **Application Load Balancer (ALB)** 가 가장 적합  
  - HTTP/HTTPS 트래픽에 특화되어 있으며,  
  - Spring 기반 REST API, 웹 애플리케이션에 최적화되어 있음
  - SSL 종료, 인증, 웹소켓, URL 라우팅 등 웹 서비스 필수 기능 지원  
  - t2.medium 인스턴스와 오토스케일링과 연동하여 무리 없이 운영 가능

---

### 요약

| 상황                      | 추천 로드 밸런서              |
|-------------------------|----------------------------|
| Spring Java 웹 애플리케이션    | Application Load Balancer (ALB) |
| 고성능 TCP/UDP 네트워크 필요    | Network Load Balancer (NLB)      |
| 네트워크 보안 가상 어플라이언스 | Gateway Load Balancer (GWLB)     |

---

## 결론:
- 아르곤2는 파라미터? 그냥 쉽게 커스텀 설정 잘못하면 보안이 오히려 떨어짐
- 차라리 파라미터값? 커스텀값? 이걸 강하게 가져가서 보안 강하게 하고
- 다중요청 공격( Dos 공격 )은 아르곤2로 커버하기 힘듦 차라리 aws 설정으로 커버할 것
- 다음 프로젝트 때 ec2가 아닌 로드 밸런서로 인스턴스 올려볼 예정

---
## 공식문서 기준 코드들
> https://github.com/RustCrypto/password-hashes/tree/master/argon2

#### 1. 파라미터 세팅
```
use argon2::{Argon2, Algorithm, Version, ParamsBuilder};

fn create_argon2_instance() -> Argon2 {
    // 메모리 사용량: 32 * 1024 KiB = 32 MiB
    let memory_kib = 32 * 1024;

    // 반복 횟수 (타임 코스트)
    let iterations = 3;

    // 병렬성 (CPU 코어 수 등)
    let parallelism = 1;

    // ParamsBuilder로 파라미터 생성
    let params = ParamsBuilder::new()
        .m_cost(memory_kib)    // 메모리 크기 (KiB 단위)
        .t_cost(iterations)    // 반복 횟수
        .p_cost(parallelism)   // 병렬성
        .build()
        .expect("Invalid Argon2 params");

    // Argon2id 알고리즘, 최신 버전 V0x13 사용
    Argon2::new(Algorithm::Argon2id, Version::V0x13, params)
}


```

#### 2. 메모리와 시간 성능 튜닝 포인트
| 파라미터     | 의미                     | 영향                                      |
| -------- | ---------------------- | --------------------------------------- |
| `m_cost` | 메모리 크기 (킬로바이트 단위)      | 클수록 메모리 사용량 증가, GPU 공격 방어력 강화, 처리 속도 감소 |
| `t_cost` | 반복 횟수 (타임 코스트)         | 클수록 연산 시간 증가, 보안성 강화                    |
| `p_cost` | 병렬 처리 수준 (CPU 코어 수 권장) | 클수록 처리 속도 향상 (병렬 처리 가능한 환경에서)           |

- m_cost는 메모리 사용량을 킬로바이트(KiB) 단위로 지정하며, 32×1024 = 32 MiB가 적당한 중간값 예시입니다.
- t_cost와 p_cost는 서버 성능과 부하 상황에 따라 적절히 조절해야 합니다.

#### 3. 권장 튜닝 방법
- 성능과 보안의 균형 맞추기
  - 서버 메모리와 CPU 리소스를 고려해 m_cost와 t_cost를 조절
  - 너무 높게 설정하면 서버 과부하 및 응답 지연 발생 가능
- 병렬성(p_cost)은 서버 CPU 코어 수에 맞게 설정
  - CPU 코어 수보다 크게 설정해도 성능 향상 제한적
- 테스트 환경에서 충분히 성능 측정 후 배포
  - 예: 메모리 32MiB, 반복 3회, 병렬 1~4 권장
- 예시
  - 메모리 64MiB, 반복 2회, 병렬 2 → 중간 보안/성능 균형
  - 메모리 16MiB, 반복 1회, 병렬 1 → 가벼운 환경용

#### 4. 해시 생성 예제

````
use argon2::{PasswordHasher};
use password_hash::SaltString;
use rand_core::OsRng;

fn hash_password(password: &str) -> String {
    let argon2 = create_argon2_instance();
    let salt = SaltString::generate(&mut OsRng);

    let password_hash = argon2.hash_password(password.as_bytes(), &salt)
        .expect("Failed to hash password")
        .to_string();

    password_hash
}

````
#### 5. 해시 검증 예제
```
use argon2::{PasswordVerifier};
use password_hash::PasswordHash;

fn verify_password(hash: &str, password: &str) -> bool {
    let argon2 = create_argon2_instance();
    let parsed_hash = PasswordHash::new(hash).expect("Invalid password hash format");

    argon2.verify_password(password.as_bytes(), &parsed_hash).is_ok()
}
```

# Argon2 비동기 처리 및 공격 대응 전략 정리

---

## 1. 비동기 처리와 서버 부하

- **동기 처리**  
  - 요청이 들어오면 바로 Argon2 해시 계산을 수행  
  - 무거운 해시 계산으로 요청이 블로킹, 동시 처리량 제한  
  - 서버 과부하 발생 위험 큼

- **비동기 처리**  
  - 해시 계산 작업을 비동기 작업 큐에 넘겨 병렬 처리  
  - 서버 블로킹 최소화, 높은 동시성 지원  
  - 구현 복잡도 상승, 응답 지연 가능성 존재

---

## 2. 스프링 환경에서 비동기 개념

- 스레드 개념보다 비동기 개념이 더 넓고 유연  
- Reactor, CompletableFuture, `@Async` 등 활용 가능  
- 비동기 처리로 CPU·메모리 자원 효율 극대화 및 확장성 확보

---

## 3. AWS 인프라와 비동기 처리의 역할 분담

| 영역         | 적용 기술 및 개념                      | 목적 및 효과                                  |
|------------|----------------------------------|-----------------------------------------|
| AWS 인프라    | 로드밸런서 + 오토스케일링             | 다중 서버에 트래픽 분산, 과부하 방지, 고가용성 확보          |
| 서버 코드    | 비동기 처리 (CompletableFuture 등)    | 무거운 작업 논블로킹 처리, 서버 블로킹 최소화, 높은 동시성 지원 |

---

## 4. 비동기 처리 시 공격자 이득 여부

- 비동기 자체가 공격자에게 특별한 이득은 아님  
- 높은 동시 요청 처리 가능 → 자원 고갈 위험 증가  
- 반드시 요청 제한, CAPTCHA, 봇 탐지 등 보안 정책 병행 필요

---

## 5. 리소스 할당량 제한 (Resource Quota Control)

- 작업 큐 크기, 스레드 풀 크기, 메모리 사용량 등 제한 필요  
- 무제한 요청 수용 시 메모리 과부하, CPU 과다 사용 위험  
- 적절한 제한 설정과 튜닝 필수  
- 너무 빡빡하면 정상 사용자 피해 발생 가능 → 균형 맞추기 중요

---

## 6. 정상 사용자 요청과 리소스 제한의 균형

- 과도한 제한은 정상 요청 차단 우려  
- 적절한 큐 크기, 스레드 수 설정 및 우선순위 관리  
- 사용자에게 재시도 안내 및 대기 메시지 제공  
- 오토스케일링과 로드밸런서로 근본적 확장성 확보

---

## 7. 공격 대응 역할 분담

| 공격 유형           | 대응 위치              | 주요 대응 방법                              |
|------------------|-------------------|----------------------------------------|
| 무작위 대입 공격      | 애플리케이션 코드 내부    | IP/계정별 요청 제한, 로그인 실패 제한, CAPTCHA, 봇 탐지       |
| DoS/DDoS 공격      | 클라우드/인프라 레벨     | AWS Shield, CloudFront, WAF, 로드밸런서, 오토스케일링          |

---

## 8. 무작위 대입 공격과 DoS 공격 비교

- 무작위 대입 공격은 한 IP 또는 소수 IP에서 반복 공격 → 코드 내 집중 방어  
- DoS/DDoS 공격은 대규모 네트워크 트래픽 공격 → 인프라 단에서 대응

---

## 9. 결론 및 추천

- 서버 안정성 위해 비동기 처리 권장  
- 공격자 이득 최소화 위해 비동기와 함께 강력한 보안 정책 병행  
- 리소스 제한은 신중히 적용하되, 우선은 요청 차단 위주 방어 설계  
- AWS 인프라(로드밸런서, 오토스케일링)로 트래픽 분산 및 확장성 확보

---

