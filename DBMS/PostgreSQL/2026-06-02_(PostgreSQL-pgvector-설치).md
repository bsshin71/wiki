# PostgreSQL pgvector 설치
- **카테고리**: #DBMS #PostgreSQL
- **태그**: #PostgreSQL #AI-RAG #pgvector #벡터DB #install
- **작성일**: 2026-06-02
- **참조 원본**: [[2026-06-02-pg_vector 설치]]

## 1. 핵심 요약
- pgvector는 PostgreSQL에서 벡터 데이터 저장·유사도 검색을 제공하는 확장. AI/RAG 파이프라인에서 임베딩 벡터 저장소로 활용.

## 2. 상세 설명

### 설치

```bash
# 소스 클론
git clone https://github.com/pgvector/pgvector.git
cd pgvector

# postgresql-devel 패키지 필요
dnf install postgresql18-devel  # RHEL/Rocky 계열

# 빌드 및 설치
make
make install  # root 권한 필요
```

### 확장 활성화

```sql
-- PostgreSQL에서 확장 설치
CREATE EXTENSION vector;

-- 설치 확인
\dx
-- vector | 0.8.2 | public | vector data type and ivfflat and hnsw access methods
```

### 기본 사용법

**벡터 테이블 생성**:
```sql
CREATE TABLE items (
    id        bigserial PRIMARY KEY,
    name      varchar(50),
    embedding vector(3)   -- 3차원 벡터
);
```

**벡터 데이터 삽입**:
```sql
INSERT INTO items (name, embedding) VALUES
    ('사과',   '[1.1, 2.2, 3.3]'),
    ('바나나', '[4.4, 5.5, 6.6]'),
    ('오렌지', '[1.0, 2.0, 3.0]');
```

**유사도 검색 (L2 거리)**:
```sql
-- [1.2, 2.4, 3.1]과 가장 가까운 3개 결과
SELECT id, name, embedding
FROM items
ORDER BY embedding <-> '[1.2, 2.4, 3.1]'
LIMIT 3;
```

**거리 연산자**:
| 연산자 | 거리 종류 |
|--------|---------|
| `<->` | L2 (유클리드) 거리 |
| `<#>` | 내적 (Inner Product) |
| `<=>` | 코사인 유사도 |

### 인덱스 (대용량 성능 최적화)

```sql
-- IVFFlat 인덱스 (근사 검색)
CREATE INDEX ON items USING ivfflat (embedding vector_l2_ops) WITH (lists = 100);

-- HNSW 인덱스 (더 빠른 검색, 높은 정확도)
CREATE INDEX ON items USING hnsw (embedding vector_l2_ops);
```

## 3. 연관 개념 (지식 연결)
- [[2026-06-02_(PostgreSQL-RPM-설치-가이드)]] — PostgreSQL 설치 환경
