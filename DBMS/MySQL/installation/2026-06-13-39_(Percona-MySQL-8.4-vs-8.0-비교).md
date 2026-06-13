# Percona MySQL 8.4 vs 8.0 비교

- **카테고리**: #DBMS #MySQL #Installation
- **태그**: #install #upgrade #version84 #호환성
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-percona mysql 8.4  vs percona mysql 8.0 - BigDataTeam]]

## 1. 핵심 요약

MySQL 8.4는 **LTS 릴리스**로 안정성이 높지만, 복제 명령어 용어 변경(`MASTER/SLAVE`→`SOURCE/REPLICA`)과 `mysql_native_password` 기본 미로드 때문에 **ProxySQL·MHA 등 이중화 도구와 호환 문제**가 발생할 수 있습니다.
따라서 현 시점에서는 ProxySQL/MHA 연동 환경에서 **8.0 최신 버전을 당분간 유지**하는 것이 안전합니다.

---

## 2. 버전 관리 모델 변경

- 8.0: 패치 릴리스에 신기능 포함 → 안정성 중시 환경에 부담.
- 8.4: **혁신 릴리스 / LTS 릴리스** 분리. 8.4는 LTS로 최소 변경·안정성 지향.

---

## 3. 추가/변경 기능

| 구분 | 내용 |
|------|------|
| UUID_VX 컴포넌트 | 다양한 버전의 UUID 생성·처리 함수 추가 |
| binlog 종속성 추적 | WRITESET 모드 내부 자료구조 개선 → 성능 향상 |
| 인증 | `mysql_native_password` 플러그인 기본 미로드 (수동 로드 필요) |
| 복제 용어 | `MASTER`/`SLAVE` → `SOURCE`/`REPLICA` 로 대체 |
| 기능 제거 | `mysqlpump`, `mysql_upgrade` 유틸리티 제거 |

---

## 4. 호환성 이슈 (⭐ 도입 판단 핵심)

### 4.1 ProxySQL 연동
- ProxySQL 2.6.0+ 부터 `caching_sha2_password` 지원 시작.
- 다만 `caching_sha2_password` 사용은 여전히 복잡.
- 8.4에서 기본 인증을 `mysql_native_password`로 변경 가능하면 ProxySQL 사용에 문제 없음 → **8.4 인증 변경 방식 확인 필요**.

### 4.2 MHA 연동
- 복제 명령어 용어 변경으로 이중화 control 도구(MHA)와 **연동 실패 가능성 매우 높음**.

복제 명령어 변경 매핑:

| 8.0 (구) | 8.4 (신) |
|----------|----------|
| START SLAVE | START REPLICA |
| STOP SLAVE | STOP REPLICA |
| SHOW SLAVE STATUS | SHOW REPLICA STATUS |
| SHOW SLAVE HOSTS | SHOW REPLICAS |
| RESET SLAVE | RESET REPLICA |
| CHANGE MASTER TO | CHANGE REPLICATION SOURCE TO |
| RESET MASTER | RESET BINARY LOGS AND GTIDS |
| SHOW MASTER STATUS | SHOW BINARY LOG STATUS |
| PURGE MASTER LOGS | PURGE BINARY LOGS |
| SHOW MASTER LOGS | SHOW BINARY LOGS |

---

## 5. 결론

> ProxySQL·MHA 이중화가 필요한 환경에서는 **8.0 최신 버전을 당분간 사용**한다.
> 8.4 전환 전 인증 방식·이중화 도구 호환성 검증이 선행되어야 한다.

---

## 6. 연관 개념

- [[2026-06-13-14_(MySQL-Percona-설치-상세-가이드)]]
- [[2026-06-13-08_(MySQL-HA-Failover-테스트-전략)]]
