# Stock System (동시성 문제 해결 중심 재고 관리 시스템)

## 프로젝트 개요

동시 요청 환경에서 재고 차감 시 발생하는 race condition 문제를 해결하는 데 초점을 둔 시스템입니다.

이 프로젝트는 비동기 처리(Kafka)가 아닌
**동시성 제어 전략 비교 및 개선 과정**에 집중합니다.

---

## 요구사항

* 재고는 0 미만으로 내려가면 안됨
* 동시에 여러 요청이 들어와도 정확한 재고 유지
* 데이터 정합성 보장

---

## 문제 상황

### 기존 구조

```text id="a9x1dp"
Client → API → DB (재고 조회 후 감소)
```

---

### 발생 문제

* race condition 발생
* 재고 초과 차감 (overselling)
* 데이터 정합성 깨짐

---

## 해결 과정

### 1. synchronized 적용

* JVM 레벨에서 동시성 제어

#### 문제점

* 단일 서버에서만 동작
* 분산 환경에서 무효

---

### 2. DB Lock (Pessimistic Lock)

```sql id="k7i1z3"
SELECT * FROM stock WHERE id = 1 FOR UPDATE;
```

#### 장점

* 데이터 정합성 보장

#### 문제점

* 성능 저하
* DB 병목 발생

---

### 3. Redis 분산 락

* Redisson 사용

#### 장점

* 분산 환경에서 동시성 제어 가능

#### 문제점

* 락 획득 비용 발생
* latency 증가

---

### 4. Redis Atomic 연산

* DECR 사용

#### 구조

```text id="k9p7yx"
Redis 재고 감소 → 성공 시 DB 반영
```

#### 장점

* atomic 보장
* 매우 빠른 처리 속도

#### 핵심

* 락 없이 동시성 해결

---

## 최종 구조

```text id="c4oq1k"
Redis → 재고 감소 (동시성 제어)
DB → 최종 저장
```

---

## 핵심 기술

* Spring Boot
* Redis
* MySQL
* JPA

---

## 핵심 설계 포인트

* 동시성 문제는 Redis에서 해결
* DB는 최종 저장소 역할
* 락 대신 atomic 연산 사용

---

## 한 줄 요약

Redis atomic 연산으로 race condition을 해결한 재고 관리 시스템

---
