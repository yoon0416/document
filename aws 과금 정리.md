# AWS 과금 정리 (프리티어인데도 과금 나오는 이유 총정리)

> - 실제 담배값 희생하며 분석한 현실적인 AWS 비용 정리입니다
> - 이건 진짜 첫배포 때나 미니 프로젝트처럼 보안, 로그 등 기타 aws 기능을 안썼다는 가정하에 작성합니다.
> - 요금은 당신들 주머니에서 나오니 정보 더블체크하고 주기적으로 aws 청구서 들어가서 어디서 과금이 나왔는지 분석하세요 제발 내 탓 ㄴ
> - AWS 요금청구서 갱신시간은 미국시간 기준입니다. 전날 일부분 요금이 다음날에 청구될 수 있습니다
---
- **추가** ebs 볼륨도 무료티어에서는 30gb까지 제공해주지만 유로 인스턴스 사용시 과금 시작합니다. 인스턴스 중지해도 과금 발생하니 참고하세요
>   - 이건 가설임 아직 청구 안나옴 알아서 확인하시길
>   - 25.05.19 인스턴스유로(중지) 후 3일간 지켜본 결과 ebs에서 청구 없음 0.00$ 확인 또한 다른 모든 곳에서 과금발생 안함 위 가설이 틀렸을 수 있음


## 1. AWS에서 과금이 발생하는 조건 총정리

| 항목 | 과금 발생 조건 | 메모 |
|------|----------------|------|
| 인스턴스 (EC2) | t2.micro는 프리티어 12개월 한정 무료 | 그 외는 유료 (t2.medium 등) |
| EBS 디스크 | 30GB 이상이면 초과 요금 발생 | EC2 중지해도 디스크는 과금됨 |
| 탄력 IP (Elastic IP) | EC2와 연결 되어 있지 않으면 유휴 요금 발생 | 연결되어 있어도 인스턴스 중지 시 과금 발생 |
| 트래픽 (Data Transfer) | 매월 아웃바운드 1GB 초과 시 과금 | 인바운드 트래픽은 무료 |
| CloudWatch 로그 | 로그 그룹 보존 기간 무제한 시 요금 발생 | 보존기간을 1일로 줄이면 무료 유지 가능 |
| 기타 서비스 (NAT, VPC, S3 등) | 숨겨진 유료 서비스 주의 | 실수로 켜진 채면 자동 과금 |

---

## 2. 인스턴스 과금 관련 요약

### 인스턴스가 프리티어라도 과금 날 수 있는 경우:
- 프리티어는 t2.micro만 해당
  → t2.medium, t3.small 등 쓰면 무조건 과금
- 무료여도 메모리 적음 (1GB) → 느려지면 터짐 → 복구 어려움
- 프리티어도 하루 10시간 이상 돌리면 트래픽 등으로 결국 과금 발생

---

## 3. 탄력 IP 과금 조건

 - AWS가 회원들에게 고정 ip를 줬는데 괘씸하게 안쓰네? 아님 한개로 부족한지 여러개 가져가네? 다른 사람들 못쓰게?
> 너 과금 ㅇㅇ

| 상황 | 과금 여부 | 설명 |
|------|-----------|------|
| 탄력 IP 1개 + EC2에 연결됨 | 무료 | 올바른 사용 |
| 탄력 IP 1개 + 연결 안 됨 | 과금 발생 | idle 상태 |
| 탄력 IP 1개 + EC2 중지 | 과금 발생 | 연결은 돼 있지만, 인스턴스 꺼짐 = idle 처리됨 |
| 탄력 IP 2개 이상 보유 | 과금 발생 | 한 개만 무료 허용 |

### 해결 방법
- 안 쓸 경우 "릴리스(Release)" 눌러 삭제
- 도메인 연결 등 꼭 필요한 경우만 사용

퍼블릭 IP는 재부팅 시 바뀌지만, 무료입니다.

---

## 4. 트래픽 과금 기준

| 항목 | 무료 한도 | 과금 조건 |
|------|-----------|-----------|
| 아웃바운드 | 1GB/월 | 초과 시 $0.12/GB |
| 인바운드 | 무제한 무료 | EC2로 들어오는 요청 (파일 업로드 등) |

### 트래픽 발생 예시

| 시나리오 | 트래픽 발생 여부 | 설명 |
|----------|------------------|------|
| 유저가 사이트 접속 | 발생 | HTML, JS, 이미지 전송 |
| 유저가 이미지 봄 | 발생 | 이미지 응답이 트래픽 |
| 유저가 글 여러 개 열람 | 발생 | 반복 전송 |
| 유저가 이미지 업로드 | 과금 없음 | 인바운드는 과금 없음 |
| EC2 내부에서 DB 조회 | 과금 없음 | 내부 트래픽은 무료 |

---

## 5. CloudWatch 로그 과금

| 항목 | 과금 조건 | 해결 방법 |
|------|------------|-----------|
| 로그 보존 기간 무제한 | 요금 발생 | 기간 1일로 변경 또는 로그 그룹 삭제 |
| 사용하지 않는 로그 그룹 | 누적 요금 발생 | 필요 없는 그룹은 반드시 삭제 |

---

## 6. 팁 요약

- 프리티어는 조건이 매우 빡셈 → 트래픽/스토리지 초과 = 즉시 과금
- 탄력 IP는 무조건 1개만 사용 + 연결된 상태 유지
- CloudWatch는 안 쓰면 삭제, 보존은 1일
- 퍼블릭 IP로도 충분한 경우가 많음
- Cost Explorer, Budget 알림 무조건 설정

---

## 7. 마지막 정리

| 할 일 | 효과 |
|--------|------|
| 인스턴스 중지 또는 삭제 | CPU/스토리지/트래픽 요금 방지 |
| 탄력 IP 릴리스 | 유휴 요금 완전 제거 |
| 이미지 압축 또는 정리 | 트래픽 감소 |
| 로그 보존 기간 조절 | CloudWatch 요금 절감 |
| 비용 알림 설정 | 예상치 못한 요금 방지 |


---
실험

# AWS 과금 실험 결론 (2025.05.17 기준)
> 이 실험은 정확한 실험이 아닐 수 있음

- 유료 인스턴스도 **중지 상태면 과금되지 않음**
- 연결된 EBS도 **30GB 프리티어 이내면 과금 없음**
- 탄력 IP는 **연결 해제된 채로 놔두면 과금됨**
- 트래픽은 월 1GB 넘으면 과금 확정
- 청구 반영은 **UTC 기준 → 한국 시간 오전 9시 이후 확인 가능**
