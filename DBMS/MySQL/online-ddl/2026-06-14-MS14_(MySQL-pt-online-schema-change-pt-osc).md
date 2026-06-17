# MySQL pt-online-schema-change (pt-osc)

- **카테고리**: #DBMS #MySQL #online-ddl
- **태그**: #MySQL #online-ddl #pt-osc #percona-toolkit #trigger #tools
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-pt-online-schema-change 사용하기 - BigDataTeam]]

## 1. 핵심 요약

pt-osc는 **트리거 기반**으로 온라인 스키마를 변경하는 Percona Toolkit 도구. ① 변경 스키마가 적용된 새 테이블 생성 → ② chunk 단위로 원본 데이터 복사(트리거로 변경분 동기화) → ③ 테이블 RENAME으로 완료. DB 부하·락 경합은 줄지만 **시간 예측이 어렵고 PK가 반드시 있어야** 한다. `--recursion-method=NONE` 필수.

---

## 2. 명령 예시

```bash
/usr/bin/pt-online-schema-change \
  --alter "MODIFY COLUMN ACC_NO char(13) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '계좌번호'" \
  D=BOSRL,t=T_DW_LM \
  --host=127.0.0.1 --port=3306 --user=root --password=**** \
  --no-drop-old-table \
  --chunk-size=10000 --chunk-size-limit=4 \
  --progress=time,30 --charset=utf8mb4 \
  --alter-foreign-keys-method=auto \
  --recursion-method=NONE \
  --preserve-triggers \
  --dry-run        # 시험가동(실제 동작 X) → 확인 후 --execute
```

> ⚠️ `--alter` 지정문자와 `\` 사이에는 **공백 한 칸** 필요.

## 3. 주요 옵션

| 옵션 | 의미 |
|------|------|
| `--recursion-method=NONE` | **필수**. 미기입 시 slave 접속 시도(호스트 오류) |
| `--chunk-size` / `--chunk-size-limit` | 복사 청크 크기·제한 |
| `--no-drop-old-table` | 구 테이블(`_t_old`) 보존 |
| `--preserve-triggers` | 기존 트리거 유지 |
| `--dry-run` / `--execute` | 시험 / 실제 실행 |

## 4. 주의점

- **PK 필수**. PK가 변경되는 트랜잭션 환경에서는 사용 주의(데이터 유실 이슈 보고됨).
- 완료 후 원본은 `_테이블_old`로 남음(`--no-drop-old-table` 시).

## 5. 연관 개념

- [[2026-06-14-MS12_(MySQL-Online-DDL-ALGORITHM-INSTANT-INPLACE-COPY)]] — 네이티브 online DDL과 비교
- [[2026-06-14-MS11_(MySQL-대용량-테이블-DDL-주의사항-실패케이스)]] — 케이스5(duplicate apply) 대안
