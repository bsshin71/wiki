# PostgreSQL 복제 구성 실습 (Streaming / Logical)

- **카테고리**: #DBMS #PostgreSQL #replication
- **태그**: #PostgreSQL #replication #pg_basebackup #publication #subscription
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Replication 테스트 - BigDataTeam]]

## 1. 핵심 요약

스트리밍 복제는 Main에 `wal_level=replica` + replication user + pg_hba 설정 후, Standby에서 **`pg_basebackup -R`** 로 데이터 복사하면 `standby.signal`·`primary_conninfo`가 자동 생성됩니다.
논리 복제는 `wal_level=logical`로 Main에 **PUBLICATION**, Standby에 테이블 생성(DDL 미복제) 후 **SUBSCRIPTION**을 만듭니다. 상태는 `pg_stat_replication`/`pg_stat_wal_receiver`로 확인합니다.

---

## 2. Streaming Replication

### Main Server
```conf
# postgresql.conf
listen_addresses = '*'
wal_level = replica
```
```conf
# pg_hba.conf
host  replication  repluser  192.168.200.22/32  scram-sha-256
```
→ replication user 생성 후 `systemctl restart postgresql-16`.

### Standby Server
```bash
systemctl stop postgresql-16
rm -rf /home/user/postgresql16/data/*
# Main의 데이터 복사 (-R: standby.signal + primary_conninfo 자동 생성)
pg_basebackup --host=192.168.200.21 --username=repluser \
  --checkpoint=fast --pgdata=.../postgresql16/data -R
systemctl start postgresql-16
```
> 성공 시 `postgresql.auto.conf`에 `primary_conninfo` 표시.

### 장기 장애 대비 (log shipping 보완)
```conf
archive_command = 'scp %p temp@192.168.200.22:/home/temp/archive-from-main/%f && pgbackrest ... archive-push %p'
```

### 상태 확인
```sql
-- Main에서 Standby 확인
SELECT * FROM pg_stat_replication;   -- state=streaming, sync_state=async
-- Standby에서 Main 확인
SELECT * FROM pg_stat_wal_receiver;  -- status=streaming
```

## 3. Logical Replication

### Main (Publisher)
```conf
wal_level = logical
```
```sql
CREATE PUBLICATION first_publication FOR TABLE contacts, repltest;
GRANT SELECT ON TABLE contacts, repltest TO repluser;
```

### Standby (Subscriber)
```bash
# DDL은 복제 안 됨 → Main에서 스키마 추출해 미리 생성
pg_dump temp -U temp -s -t repltest
```
```sql
-- 추출한 CREATE TABLE 실행 후
CREATE SUBSCRIPTION my_subscription
  CONNECTION 'dbname=temp host=192.168.200.21 port=5432 user=repluser password=replication'
  PUBLICATION first_publication;
```

## 4. 연관 개념

- [[2026-06-14-PG15_(PostgreSQL-복제-방식-물리-논리)]]
- [[2026-06-14-PG14_(PostgreSQL-pg_dump-테이블-DDL-추출)]]
- [[2026-06-02_(PostgreSQL-pgBackRest-백업-가이드)]]
