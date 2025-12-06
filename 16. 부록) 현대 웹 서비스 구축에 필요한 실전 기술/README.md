# 16장. 부록) 현대 웹 서비스 구축에 필요한 실전 기술

## 강의 1: 작업큐(Job Queue) 시스템

비동기 처리 기반 확장성 확보

### 웹 서비스와 비동기 처리 필요성

- 동기 요청 증가 → API 서버 응답이 느려짐
- 메일 발송·이미지 처리 등은 비동기 처리 적합
- 요청을 큐에 넣고 Worker가 백그라운드 처리 → 응답 속도 향상

### 작업큐 기본 구성

<img width="688" height="406" alt="image" src="https://github.com/user-attachments/assets/e6588ff4-ecd9-4542-916a-b4c3c57cca4f" />

- 클라이언트: 작업 요청을 큐에 삽입
- 큐 서버: 작업 목록 저장
- Worker: 큐에서 작업 꺼내 실행

### TheSchwartz

- MySQL과 같은 RDBMS를 사용하는 작업큐 시스템
- RDMBS로 작업큐를 관리함으로써 높은 신뢰성과 안정성 확보
- But, 대량 처리 속도는 다소 늦음
- 따라서, TheSchwartz에서 다루는 작업의 크기는 어느 정도 크게 하는 편이 좋다.

### Gearman

- DB 의존 없이 작업의 정보를 메모리에 저장함으로써 성능을 높인 경량/고속 작업큐
- 병렬 처리·실시간 처리에 강함

## 강의 2: 스토리지 선택

### 웹 서비스와 데이터 특성

- 데이터는 시간이 지날수록 누적 증가(TB → PB)
- 읽기·쓰기 비율, 조회 패턴, 일관성 요구 등 애플리케이션 특성이 제각각
- 구조적 데이터(사용자/글), 반구조적 데이터(로그), 대용량 파일 등 **데이터 형태가 다양**
- 하나의 스토리지 타입으로 모든 요구를 만족시키기 어려움

<img width="695" height="236" alt="image" src="https://github.com/user-attachments/assets/cc9e2afe-0554-4dce-a2f4-d3a6cc3a3e68" />

### 왜 여러 스토리지를 병행해야 하나

- **트랜잭션/검색/정렬**은 RDBMS가 강함
- **읽기 많은 단순 Key 기반 조회**는 KV 스토어가 압도적
- **대용량 파일 저장**은 분산 파일 시스템이 적합
- **분석 목적 로그**는 HDFS 같은 분석 스토리지가 필요
    
    → 데이터 성격별로 스토리지를 나눠 쓰는 것이 현대 서비스의 기본
    

### 적절한 스토리지 선택의 어려움

- 데이터 성격이 다양해 범용 스토리지가 없음
- 서비스가 성장하면서 접근 패턴이 자주 변함(검색 증가, 조회 폭증 등)
- 성능·비용·운영 복잡성 사이에서 완전한 정답이 없음
- 데이터가 커질수록 기존 구조는 쉽게 병목 → 중간에 구조 변경 비용이 큼

### 데이터가 계속 증가하는 환경에서의 선택 기준

- 평균/최대 크기
- 읽기/쓰기 비율
- 조회 패턴
- 장애 허용 범위
- 비용/SSD 여부
- 일관성 요구 수준
    
    → 서비스 특성에 맞는 저장소 조합이 핵심
    

### RDBMS(MySQL 중심)

- SQL 기반, 강한 일관성, 조인 지원
- 다수의 웹 서비스가 기본 선택
- MySQL 엔진 비교
    - MyISAM: 빠르지만 트랜잭션 없음, Table Lock 기반
    - InnoDB: 트랜잭션·Row Lock 지원 → 기본 선택
    - Maria 계열: MyISAM 개선형

### 분산 Key–Value 스토어

- 단순 key/value 저장 → 고성능·확장성
- 캐시·세션·단순 스토어 등에 적합

### memcached

- 메모리 기반 분산 캐시
- Consistent Hashing 지원
- 읽기 많은 서비스에 탁월
- 휘발성 데이터 → 영속성 없음

### TokyoTyrant

- TokyoCabinet 기반 영속 key-value
- 대량 저장 + 고성능 병행
- 파일 기반 Kvstore - 디스크에 데이터를 기록함으로써 영속화 가능
- memcached와 비교하면 디스크 액세스가 발생하는 만큼 성능은 약간 떨어지지만, RDBMS와  비교하면 매우 빠르다.

### 분산 파일 시스템

- 대용량 파일 저장
- 하테나 사용: MogileFS
- 기타: NFS, DRBD, HDFS

### MogileFS

- 파일을 여러 스토리지 서버에 자동 분산
- 장애 내구성 우수
- 메타데이터는 DB에서 관리

### HDFS(Hadoop)

- 대용량 분석 중심
- MapReduce와 조합해 로그 분석 등에 활용

### 스토리지 선택전략

<img width="689" height="439" alt="image" src="https://github.com/user-attachments/assets/a49a305d-7029-47f7-9795-3fda4a63fc63" />

## 강의 3: 캐시 시스템

### 캐시 도입 이유

- API 서버·DB 서버 부하 급증
- 리버스 프록시 기반 캐시는 단순하면서 효과적인 부하 감소책
- 대표 구성: 클라이언트 → 리버스 프록시 → 캐시 서버 → AP서버

### Squid

전통적·안정적 캐시 서버

<img width="693" height="215" alt="image" src="https://github.com/user-attachments/assets/652c1455-e2c2-49e1-9130-f3d24d0cd7c9" />

### 특징

- HTTP/HTTPS/FTP 지원
- 캐시 서버를 여러 대 묶을 수 있음(ICP, CARP)
- AP 서버 앞단에서 응답을 캐싱 → 부하 감소
- 2단 구성(CARP)으로 확장 가능

### 주의점

- 초기 COSS 할당 비용 큼
- 디스크 I/O 의존성
- 설정 복잡
- 장애 시 캐시 손실 가능성 있음

### Varnish

고성능 캐시 서버(현대적 구조)

### 장점

- 메모리 기반 mmap 구조 → 고속
- 설정 언어(VCL)로 고도화 가능
- Squid 대비 I/O가 적어 고성능
- 트래픽 피크에 강함
- 캐시 객체를 빠르게 재조립하여 응답

### 운영 포인트

- VCL 설정이 매우 중요
- 필요 시 varnishlog/varnishncsa로 로그 추적
- Squid 대비 캐시 적중률은 비슷하지만 처리 성능이 더 높음

## 강의 4: 계산 클러스터(Hadoop)

대량 로그 데이터 분석을 위한 병렬 처리 기술

### 대규모 로그 데이터 문제

- 대형 서비스는 하루 수십~수백 GB 로그 발생
- 단일 서버 분석 불가 → 병렬 처리 시스템 필요
- 하테나: 약 47GB 로그를 Hadoop으로 분석

### MapReduce 개념

- Map: 입력 데이터 분할·가공
- Shuffle: key 기준으로 재배열
- Reduce: 최종 결과 생성
- 단순하지만 대량 병렬 처리에 매우 적합

### MapReduce 처리 흐름

- (k1, v1) 목록 입력
- Map → (k2, v2)
- Shuffle → key=k2 기준 그룹화
- Reduce → (k2, list(v3))
- 결과 리턴

<img width="690" height="383" alt="image" src="https://github.com/user-attachments/assets/8d729502-251f-471b-9254-c342c48777fa" />

### Hadoop

- 오픈소스 MapReduce 구현체
- HDFS 기반 분산 파일 저장
- Yahoo, Facebook 등에서 실제 사용
- 한 작업에서 수천 개의 Map/Reduce 태스크가 병렬 실행
- 하테나 사례:
    - 4,303개 Map 태스크
    - 하나의 Reduce 태스크
    - 47GB 로그 분석
