# MySQL binlog Archiver Daemon 스크립트

- **카테고리**: #DBMS #MySQL #binlog
- **태그**: #MySQL #binlog #archiving #daemon #systemd #script
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-binlog archiver daemon - BigDataTeam]]

## 1. 핵심 요약

`mysqlbinlog --stop-never`를 **백그라운드 데몬**으로 실행해 binlog를 실시간 아카이빙한다. 시작 스크립트는 마지막 아카이브 파일 기준으로 재개 지점을 자동 결정하고 PID를 파일로 저장한다. `systemctl`로 서비스 등록해 시스템 재시작 시 자동 구동한다.

---

## 2. Start Script 핵심 로직

```bash
# 마지막 아카이브 파일 기준으로 시작 binlog 결정
get_last_archlog() {
  find $ARCHDIR -name "bin.*" -type f | sort | tail -n 2 | head -n 1
}
LAST_ARCHLOG=$(basename $(get_last_archlog))
BINLOG_LIST=$(mysql -u$USER -p$PASS -e "SHOW BINARY LOGS;" 2>/dev/null | awk 'NR>1 {print $1}')

# LAST_ARCHLOG와 매칭하는 binlog 찾기 (없으면 가장 오래된 것부터)
for BINLOG in $BINLOG_LIST; do
  if [[ "$LAST_ARCHLOG" == "$BINLOG" ]]; then START_BINLOG="$BINLOG"; break; fi
done
[[ -z "$START_BINLOG" ]] && START_BINLOG=$(echo "$BINLOG_LIST" | head -n 1)

# 백그라운드 실행
nohup mysqlbinlog --read-from-remote-server --raw --stop-never \
  --host=$MYSQL_HOST --port=$MYSQL_PORT \
  --user=$REP_USER --password=$REP_PASS \
  --result-file="$ARCHDIR/" "$START_BINLOG" 2>&1 | grep -v "Warning" >> "$ARCH_TRCLOG" &
echo $! > "$TRCDIR/mysqlbinlog.pid"
```

## 3. Stop Script

```bash
PID=$(cat "$TRCDIR/mysqlbinlog.pid")
kill $PID && rm -f "$TRCDIR/mysqlbinlog.pid"
```

## 4. systemd 서비스 등록

```bash
systemctl daemon-reload
systemctl start mysqlbinlog-archiver
systemctl status mysqlbinlog-archiver
```

> DB 생존 여부 확인: `mysqladmin ping | grep alive`로 DB 다운 시 archiver 미시작.

## 5. 연관 개념

- [[2026-06-14-MS16_(MySQL-binlog-아카이빙-방식-비교-cp-vs-mysqlbinlog)]]
- [[2026-06-14-MS18_(MySQL-mysqlbinlog-유틸-사용법)]]
