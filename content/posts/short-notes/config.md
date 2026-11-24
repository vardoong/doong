---
date: '2026-11-24T13:50:42+09:00'
draft: false
title: "짧은 노트 - 설정"
---

# 경고

## 1. HikariPool-1 - Retrograde clock change detected, soft-evicting connections from pool

### 해결
로컬 -> 무시

### 전체 경고 내용

```text
2026-11-24T13:40:42.673+09:00  WARN 56464 --- [mini-quests] [l-1:housekeeper] com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Retrograde clock change detected (housekeeper delta=28s21ms), soft-evicting connections from pool.
```

### 상황
서버 실행시 간헐적으로 발생.

### 의미

HikariCP(Spring Boot의 기본 커넥션 풀)에는 housekeeper라는 백그라운드 스레드가 있어요. 이 스레드가 30초마다 깨어나서 오래된 커넥션을 정리하는데, 깨어날 때마다 "지난번 이후로 시간이 얼마나 흘렀나"를 확인해요.

정상: 약 30초 간격
이번 경우: delta=28s21ms, 즉 약 28초. 예상보다 짧아서 시계가 뒤로 갔다고 판단한 거예요.

시계가 믿을 수 없는 상태라고 보면 커넥션의 수명 계산도 틀어질 수 있으니, 풀 안의 기존 커넥션을 **soft-evict(부드럽게 퇴출)** 해요. 사용 중인 커넥션은 건드리지 않고, 놀고 있는 커넥션은 닫고, 사용 중인 것은 반납될 때 닫아서 새 커넥션으로 교체하는 방식이에요.

### 발생 이유 

**로컬 개발 환경에서 흔한 원인**

- 노트북 절전/잠자기 후 복귀: 가장 흔해요. 덮개를 닫았다가 열었을 때 자주 떠요.

- 시간 동기화(NTP): OS가 시계를 자동으로 보정하면서 몇 초 조정된 경우

- 가상머신/WSL/Docker: 호스트와 게스트의 시계가 어긋났다가 맞춰질 때

- 디버거로 breakpoint에 걸려서 멈춰 있다가 재개한 경우: 스레드가 멈춰 있어서 시간 간격이 어긋나요

## 2. spring.jpa.open-in-ivew dafult

### 해결
`application.properties`:
`spring.jpa.open-in-view` 값을 false 로 설정

- false 로 설정하면
  - 서비스 계층(트랜잭션)이 끝나는 순간 DB 커넥션 반납
  - 엔티티는 트랜잭션 내에서 DTO 로 변환해줘야함

### 전체 경고 내용

```text
2026-11-24T13:34:11.801+09:00  WARN 54408 --- [mini-quests] [           main] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
```

### 발생 이유
- `spring.jpa.open-in-view` 값이 default 로 true 임.
- true 인 경우
  - http 요청이 들어와서 응답이 나갈 때까지 DB 커넥션과 JPA 영속성 컨텍스트가 계속 열려 있음
  - 그래서 서비스 계층이 끝난 뒤 뷰나 컨트롤러에서 Lazy Loading 을 해도 예외(LazyInitializationException)가 발생하지 않음
  - DB 커넥션을 오래 붙잡고 있어서, 트래픽이 많으면 커넥션 풀이 고갈되기 쉬움
  - 뷰 렌더링 또는 엔티티를 JSON으로 변환하는 중에 쿼리가 나가서, (N+1 같은) 성능 이슈를  문제를 탐지하기 어려움
