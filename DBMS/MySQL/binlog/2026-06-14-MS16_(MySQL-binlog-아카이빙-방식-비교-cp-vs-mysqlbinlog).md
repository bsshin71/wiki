# MySQL binlog 아카이빙 방식 비교 (cp vs mysqlbinlog)

- **카테고리**: #DBMS #MySQL #binlog
- **태그**: #MySQL #binlog #archiving #PITR #backup
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-binlog 아카이빙 방식 선정 - BigDataTeam]]

## 1. 핵심 요약

PITR(Point-in-Time Recovery)를 위한 binlog 아카이빙 방식으로 **① OS `cp` 주기적 복사** 와 **② `mysqlbinlog --stop-never` 실시간 스트리밍** 두 가지가 있다. 단순 로컬 백업은 cp, **실시간·원격·고가용성 환경에서는 mysqlbinlog** 방식이 적합하다.

---

## 2. 방식별 비교

| 항목 | cp 방식 | mysqlbinlog 방식 |
|------|---------|-----------------|
| 실시간 복제 | ❌ 불가 | ✅ (`--stop-never`) |
| 원격 복제 | ❌ 동일 서버만 | ✅ (`--host` 옵션) |
| 데이터 일관성 | 중간(복사 충돌 가능) | 높음(MySQL 공식 복사) |
| 데이터 손실 가능성 | 있음(타이밍 의존) | 거의 없음 |
| CPU 사용량 | 낮음 | 높음(네트워크·스트리밍) |
| 운영 복잡도 | 쉬움 | 백그라운드 프로세스 관리 필요 |
| 설치/설정 | 단순 | 설정 필요 |

## 3. cp 방식 스크립트 (로컬 주기적 복사)

```bash
BINLOG_LIST=$(mysql -u$USER -p$PASS -e "show binary logs" 2>/dev/null | awk '{if(NR>=2) print $1}')
while read binlogfile; do
  orgfsize=$(stat -c%s ${mysqlbinlog_dir}/${binlogfile})
  archfsize=$(stat -c%s ${ARCHDIR}/arch.${binlogfile} 2>/dev/null || echo 0)
  if [ "$archfsize" == "0" ] || [ $orgfsize -gt $archfsize ]; then
    cp ${mysqlbinlog_dir}/${binlogfile} ${ARCHDIR}/arch.${binlogfile}
  fi
done <<< "$BINLOG_LIST"
```

## 4. mysqlbinlog 방식 (실시간 스트리밍)

```bash
mysqlbinlog --read-from-remote-server --raw --stop-never \
  --host=127.0.0.1 --port=3306 --user=root --password=**** \
  --result-file=/mnt/dbbackup/prd/archbin/ \
  bin.000156
```

## 5. 연관 개념

- [[2026-06-14-MS17_(MySQL-binlog-archiver-daemon-스크립트)]] — mysqlbinlog 방식 데몬화
- [[2026-06-14-MS18_(MySQL-mysqlbinlog-유틸-사용법)]]
- [[2026-06-13-44_(MySQL-시점-복구-PITR)]]
