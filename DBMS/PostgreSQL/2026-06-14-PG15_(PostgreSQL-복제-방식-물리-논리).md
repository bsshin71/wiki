# PostgreSQL 복제 방식 (물리·논리)

- **카테고리**: #DBMS #PostgreSQL #replication
- **태그**: #PostgreSQL #replication #WAL #streaming #logical
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Postgres replication - BigDataTeam]]

## 1. 핵심 요약

PostgreSQL 복제는 **물리 복제(로그 전달/스트리밍)** 와 **논리 복제(Publication/Subscription)** 로 나뉩니다.
물리 복제는 WAL을 통째로 전달(읽기 전용 Standby), 논리 복제는 **테이블 단위 선택 복제 + Subscriber 쓰기 + 이기종/버전 간 복제**가 가능합니다.

---

## 2. 로그 전달 방식 (Log shipping)

- Main의 완성된 **WAL 파일(16MB)** 을 통째로 Standby로 전송(SCP 등).
- 파일이 다 채워져야 전달 → **트래픽 적으면 동기화 지연**, 채워지던 WAL은 장애 시 유실 위험.
- 모드: **Hot Standby**(SELECT 가능) / **Warm Standby**(접속·SELECT 불가, 거의 미사용).
- 파라미터: Main `wal_level`·`archive_mode`·`archive_command`·`archive_timeout`·`max_wal_senders` / Standby `restore_command`·`hot_standby`·`archive_cleanup_command`.

## 3. 스트리밍 방식 (Streaming)

- WAL을 파일 단위가 아닌 **WAL Record 단위로 실시간 전송**.
- Main `WAL Sender` ↔ Standby `WAL Receiver` 프로세스 통신. 기본 **비동기**(커밋 후 ~1초 미만 지연).
- ⚠️ Standby 장기 장애 시 Main이 WAL 순환 재사용 → 로그 덮어쓰기로 복제 단절 → `wal_keep_size`(구 `wal_keep_segments`)로 보관량 확보.
- Log shipping과 **하이브리드 구성** 가능.

## 4. 논리 복제 (Logical Replication)

### 물리 복제의 한계 (도입 배경)
- 일부 테이블/DB만 선택 복제 불가, Standby 로컬 쓰기 불가, **Major 버전·OS/CPU 다르면 복제 불가**.

### 장점
- **선택적 복제**(특정 DB/Table), Subscriber **쓰기 허용**, **이기종/버전 간 복제**(무중단 마이그레이션·업그레이드), Cascade 다중 구성, DML 유형 필터.

### 제약
- Publication↔Subscription **테이블명 일치 필수**, **PK/Unique 필수**(replica identity), 기본 단방향.
- **미지원 객체**: Sequence, Large Object, Materialized View, Partition Root Table, Foreign Table.
- **DDL 미복제**(각 서버 수동 수행), 제약 충돌 시 복제 즉시 중단(수동 해결 필요).

## 5. 연관 개념

- [[2026-06-14-PG16_(PostgreSQL-복제-구성-실습-Streaming-Logical)]]
- [[2026-06-14-PG18_(PostgreSQL-Patroni-HA-구성)]]
