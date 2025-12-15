
## 1. RDB vs NoSQL — 왜 나뉘었나?

### RDB
- **목적**: 데이터 무결성과 정합성 보장
- **핵심 수단**
  - 스키마 (PK, FK, 제약조건)
  - 트랜잭션 + ACID
- **강점**
  - 정확성, 신뢰성
  - 복잡한 비즈니스 로직 처리
- **한계**
  - 수평 확장 어려움
  - 대규모 트래픽·초저지연 환경에서 병목

### NoSQL
- **목적**: 확장성, 성능, 유연성
- **특징**
  - 스키마 유연
  - 분산 환경 전제
  - 정합성 일부 포기
- **주요 사용처**
  - 캐시, 세션, 로그
  - 실시간 조회
  - 대량 쓰기 / 이벤트 스트림

> NoSQL은 RDB의 대체가 아니라 **보완재**

---

## 2. ACID — RDB가 무결성을 지키는 이유

ACID는 **트랜잭션의 성질**을 설명한다.

| 속성 | 의미 | 핵심 메커니즘 |
|---|---|---|
| Atomicity | 전부 성공 or 전부 실패 | Undo Log |
| Consistency | 무결성 규칙 유지 | PK/FK/Constraint |
| Isolation | 동시성 간섭 차단 | Lock, MVCC |
| Durability | 커밋 데이터 영속 | WAL(Redo Log) |

---

## 3. Atomicity & Durability — 로그의 역할

### Undo Log
- 변경 전 데이터 저장
- Rollback 및 장애 복구 목적

### Redo Log (WAL)
- 변경 사항을 먼저 디스크에 기록
- 장애 발생 시 재적용(replay)

> Undo / Redo는 **데이터가 아닌 차이(delta) 로그**
- 순차 기록 가능
- 압축 효율 높음
- 디스크 I/O 최적화

---

## 4. Isolation — 동시성 문제

### 대표적인 이상 현상
1. Dirty Read
2. Non-Repeatable Read
3. Phantom Read
4. Lost Update

### 해결 방식
- Lock 기반
- MVCC 기반

---

## 5. Lock — 격리성을 강제로 보장

### 공유 락 (S-Lock)
- 읽기 허용 / 쓰기 차단
- 여러 트랜잭션 동시 읽기 가능
- 인덱스 엔트리(레코드 주소)에 락

### 배타 락 (X-Lock)
- 읽기 + 쓰기 모두 차단
- 자원 독점
- 성능 비용 큼 → 실무에서는 최소 사용

### 갭 락 (Gap Lock)
- **존재하지 않는 레코드 사이 공간을 잠금**
- 팬텀 리드 방지 목적
- 락 범위가 넓어 **데드락 위험**

---

## 6. MVCC — 락 없이 읽는 이유

- **읽기**: 과거 스냅샷
- **쓰기**: 새로운 버전 생성
- 읽기/쓰기 충돌 제거
- 일반 SELECT는 락 없음
- `FOR UPDATE` 등은 현재 버전 + 락 사용

> 현대 RDB 성능의 핵심 메커니즘

---

## 7. CAP 이론 — NoSQL 선택 기준

- **C (Consistency)**: 항상 같은 값
- **A (Availability)**: 항상 응답
- **P (Partition Tolerance)**: 네트워크 분리 허용

> 분산 환경에서는 **P는 필수**
→ **CP 또는 AP 선택**

---

## 8. PACELC — 현실적인 분산 시스템 이론

- **P 상황**: Consistency vs Availability
- **Else (정상 상황)**:
  - Latency vs Consistency

> 실제 NoSQL은 지연과 일관성의 트레이드오프를 설계로 선택

---

## 9. 확장 전략

### Scale Up
- 서버 성능 증설
- 한계 명확

### Scale Out
- 서버 수 증가
- 분산 구조 필요

---

## 10. Replication / Sharding / Partitioning

### Replication
- 동일 데이터 복제
- 목적: 가용성 + 읽기 성능
- 쓰기는 Primary 한 곳
- Replication Lag = 실시간성 포기

### Sharding
- 데이터를 여러 DB로 분산 저장
- **쓰기 분산 목적**
- 샤딩 키 = 라우팅 기준
- 운영 난이도 매우 높음
- Scale-out의 마지막 카드

### Partitioning
- 단일 DB 내부에서 테이블 분할
- 목적: 성능 최적화
- Partition Key로 불필요한 데이터 접근 제거
- Scale-up 개념

---

## 11. 언어는 DB를 어떻게 지원하나?

### 핵심 원칙
> 언어는 DB를 직접 제어하지 않는다.

### 구성 요소

#### DB Driver
- 네트워크 통신
- SQL 전달
- 결과 파싱

#### 트랜잭션 API
- 트랜잭션 경계만 선언
- 실제 ACID 처리는 DB 담당

#### 추상화 계층 (ORM)
- 개발 편의성 제공
- 성능·정합성은 DB 책임

> 언어가 DB를 지원한다는 의미는  
> **DB와 대화할 수 있는 표준 인터페이스를 제공한다는 뜻**

---

## 최종 한 문장 요약

> **RDB는 정확성을, NoSQL은 확장성을 선택한 결과물이며  
> 트랜잭션·락·MVCC·샤딩은 모두 ‘정합성과 성능의 트레이드오프’ 위에 존재한다.**
