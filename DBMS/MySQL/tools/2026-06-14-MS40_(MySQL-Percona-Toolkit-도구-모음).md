# MySQL Percona Toolkit 도구 모음

- **카테고리**: #DBMS #MySQL #tools
- **태그**: #MySQL #tools #percona-toolkit #pt-query-digest #pt-summary #monitoring #분석
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Percona-Toolkit - BigDataTeam]]

## 1. 핵심 요약

Percona Toolkit은 MySQL DBA를 위한 고급 CLI 도구 모음. `pt-query-digest`(슬로우로그·프로세스리스트·tcpdump 쿼리 분석), `pt-summary`(시스템 요약), `pt-mysql-summary`(MySQL 상태 요약), `pt-archiver`(행 아카이브), `pt-diskstats`(디스크 I/O 모니터링) 등이 포함된다.

---

## 2. 설치

```bash
sudo dnf install percona-toolkit   # RHEL/CentOS
# 또는 percona.com에서 RPM 다운로드 후 설치
```

## 3. 주요 도구

### pt-query-digest — 쿼리 분석

```bash
# slow log 분석
pt-query-digest --type='slowlog' mysql-query.log > slow_stat.log

# processlist 실시간 분석
pt-query-digest --processlist h=localhost,u=root --ask-pass --interval 1 --output slowlog > process.log

# tcpdump 기반 분석
tcpdump -s 65535 -x -nn -q -tttt -i any port 3306 > mysql.tcp.txt
pt-query-digest --limit=100% --type tcpdump mysql.tcp.txt > ptqd_tcp.out
```
출력: 쿼리별 총 실행시간·호출수·avg/max/95th·I/O 통계.

### pt-summary — 시스템 요약
```bash
pt-summary
# CPU·메모리·디스크·네트워크·프로세스 현황 출력
```

### pt-mysql-summary — MySQL 요약
```bash
pt-mysql-summary --user=root
# InnoDB 버퍼풀·복제 상태·processlist·Status Counters 요약
```

### pt-archiver — 행 아카이브
```bash
pt-archiver --source h=127.0.0.1,D=sourcedb,t=T_DEAL_LM \
  --file 'archive_%Y-%m-%d' --where "1=1" -uroot --no-check-charset
# 조건부 row를 파일로 export → LOAD DATA INFILE로 재입력 가능
```

### pt-align — 출력 정렬
```bash
cat a.txt | pt-align   # 컬럼 정렬 출력
```

### pt-diskstats — 디스크 I/O
```bash
pt-diskstats --interval 5   # 주의: 일부 환경에서 hang 보고됨
```

## 4. 연관 개념

- [[2026-06-14-MS14_(MySQL-pt-online-schema-change-pt-osc)]] — pt-osc
- [[2026-06-14-MS39_(MySQL-pt-table-sync-테이블-데이터-동기화)]] — pt-table-sync
- [[2026-06-14-MS38_(MySQL-LOAD-DATA-LOCAL-INFILE-파일-임포트)]] — pt-archiver 연계
