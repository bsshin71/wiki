# MySQL InnoDB Storage Engine 아키텍처

- **카테고리**: #DBMS #MySQL #innodb
- **태그**: #MySQL #innodb #buffer-pool #tablespace #doublewrite #아키텍처
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-InnoDB storage engine - BigDataTeam]]

## 1. 핵심 요약

InnoDB 구조는 **In-Memory(Buffer Pool·Change Buffer·AHI·Log Buffer)** 와 **On-Disk(Tables·Indexes·Tablespaces·Doublewrite·Redo/Undo Log)** 로 나뉜다. Buffer Pool은 물리메모리 60~80%를 차지하며 LRU(new 5/8·old 3/8) 서브리스트로 관리되고, Clustered Index(=PK)와 Secondary Index(PK 컬럼 포함)로 데이터를 조직한다. 각 구조의 상세는 개별 문서로 분리.

---

## 2. In-Memory 구조

| 구조 | 요점 |
|------|------|
| **Buffer Pool** | 데이터·인덱스 캐시. LRU(new 5/8 / old 3/8). mysqldump·full scan이 오염시킴 → midpoint insertion |
| **Change Buffer** | secondary index 변경을 모아 지연 디스크 반영(buffer pool 25%, 최대 50%). SSD·풀캐시 시 비활성 권장 |
| **Adaptive Hash Index** | 자주 쓰는 키 자동 해시화(1/64) → [[2026-06-14-MS01_(MySQL-InnoDB-Adaptive-Hash-Index-AHI)]] |
| **Log Buffer** | redo log 디스크 기록 전 버퍼(기본 16M, `innodb_log_buffer_size`) |

## 3. On-Disk 구조

### 인덱스
- **Clustered Index** = PK(없으면 NOT NULL UNIQUE → auto-inc → 6byte row id). 짧게 구성할수록 유리.
- **Secondary Index**: clustered 컬럼 포함, B-Tree(leaf=16KB `innodb_page_size`). `innodb_fill_factor`로 유휴 공간.

### Tablespaces
| 유형 | 설명 |
|------|------|
| System | change buffer 등 |
| **File-Per-Table** (`innodb_file_per_table=ON`) | 테이블=1파일(`t1.ibd`). truncate/drop로 OS 공간 반환, export/import 용이 |
| General | `CREATE TABLESPACE` 공유 |
| Undo | clustered index 변경 취소 정보(`innodb_max_undo_log_size`) |
| Temporary | session/global temp |

### Doublewrite Buffer
- 페이지 플러시 전 사본 기록 → partial write 장애 복구. 8.0.20+ 별도 파일.

### Redo Log
- `ib_logfile0/1` circular. 그룹 커밋, 8.0.17+ archiving, 대량 로딩 시 `ALTER INSTANCE DISABLE INNODB REDO_LOG`.

## 4. 연관 개념

- [[2026-06-14-MS01_(MySQL-InnoDB-Adaptive-Hash-Index-AHI)]]
- [[2026-06-13-26_(MySQL-Redo-Log-및-로그버퍼-설정)]] · [[2026-06-13-27_(MySQL-Redo-Log-아카이빙)]] · [[2026-06-13-29_(MySQL-Undo-로그-관리)]] · [[2026-06-13-30_(MySQL-Change-Buffer)]]
- [[2026-06-14-MS03_(MySQL-InnoDB-Locking-Transaction-Model)]]
