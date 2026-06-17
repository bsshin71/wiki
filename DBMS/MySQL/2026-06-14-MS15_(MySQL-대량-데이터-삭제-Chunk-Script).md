# MySQL 대량 데이터 삭제 (Chunk Script)

- **카테고리**: #DBMS #MySQL #admin
- **태그**: #MySQL #admin #대용량작업 #delete #chunk #bash-script
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-대량데이터삭제용 script - BigDataTeam]]

## 1. 핵심 요약

대량 DELETE를 한 번에 수행하면 긴 트랜잭션·락·복제지연을 유발하므로, **`LIMIT N`으로 청크 단위 반복 삭제 + 짧은 sleep**으로 부하를 분산한다. `ROW_COUNT()`로 삭제 행 수를 확인해 0이 될 때까지 반복하며, 심볼/키별로 루프를 돌린다.

---

## 2. 패턴

```bash
#!/bin/bash
export MYSQL_PWD="****"
MYCMD="mysql -uroot -A *** -sN "
DELLIMIT=50000      # 청크 크기
SLEEP=0.1           # 청크 간 대기(부하 분산)

run_delete_dml() {   # 청크 1회 삭제 + 삭제행수 반환
  $MYCMD <<EOF
delete from MSR_DOMINANCE where symbol='${1}' and unix_time < unix_timestamp('2025-01-01') limit ${DELLIMIT};
select row_count();
EOF
}

run_delete() {       # 한 심볼을 0건이 될 때까지 반복 삭제
  delcnt=1
  while [ $delcnt -gt 0 ]; do
    delcnt=`run_delete_dml ${1}`
    [ $delcnt -lt 1 ] && break
    sleep $SLEEP
  done
}

for SYB in BNB BTC EOS ETH XRP; do run_delete "${SYB}"; done
```

## 3. 핵심 포인트

| 요소 | 이유 |
|------|------|
| `LIMIT 50000` | 한 트랜잭션을 작게 → 락 보유·undo·binlog 부담 감소 |
| `sleep 0.1` | 복제 지연·디스크 부하 완화 |
| `row_count()` 루프 | 남은 데이터 0까지 자동 반복, 진행 로그 출력 |
| 인덱스 활용 | `symbol`,`unix_time` 조건이 인덱스를 타야 효율적 |

## 4. 연관 개념

- [[2026-06-14-MS11_(MySQL-대용량-테이블-DDL-주의사항-실패케이스)]]
- [[2026-06-14-MS14_(MySQL-pt-online-schema-change-pt-osc)]]
