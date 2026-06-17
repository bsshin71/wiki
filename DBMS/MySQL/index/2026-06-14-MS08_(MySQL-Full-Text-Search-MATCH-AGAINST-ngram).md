# MySQL Full-Text Search (MATCH AGAINST · ngram)

- **카테고리**: #DBMS #MySQL #index
- **태그**: #MySQL #index #fts #full-text #MATCH-AGAINST #ngram #한글검색
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Full text search 테스트 - BigDataTeam]]

## 1. 핵심 요약

Full-Text Index는 `LIKE '%검색어%'`보다 빠른 단어 단위·가중치 텍스트 검색. **InnoDB + CHAR/VARCHAR/TEXT + utf8mb4** 에 `FULLTEXT KEY` 생성 후 `MATCH(col) AGAINST(...)`로 검색하며, 자연어 모드와 **BOOLEAN MODE(실무 다수)** 가 있다. MySQL FTS는 **형태소 미지원**이라 한글은 **N-gram 파서**(`WITH PARSER ngram`)를 사용한다.

---

## 2. 인덱스 생성·검색

```sql
CREATE TABLE articles (
  id INT AUTO_INCREMENT PRIMARY KEY,
  title VARCHAR(200), content TEXT,
  FULLTEXT KEY ft_title_content (title, content)
) ENGINE=InnoDB;

-- 자연어 모드(기본): score 자동 계산, 불용어·최소길이 적용
SELECT * FROM articles WHERE MATCH(title,content) AGAINST('mysql performance');
SELECT id, MATCH(title,content) AGAINST('mysql') AS score FROM articles ORDER BY score DESC;
```

## 3. BOOLEAN MODE 연산자

| 연산자 | 의미 | | 연산자 | 의미 |
|--------|------|---|--------|------|
| `+` | 반드시 포함 | | `"` | 구문 검색 |
| `-` | 제외 | | `()` | 그룹 |
| `*` | 와일드카드 | | `> <` | 가중치 증감 |

```sql
SELECT * FROM articles WHERE MATCH(title,content)
AGAINST('+mysql -oracle' IN BOOLEAN MODE);
```

## 4. 한글 검색 — N-gram 파서

```sql
CREATE TABLE posts (
  id INT PRIMARY KEY AUTO_INCREMENT, content TEXT,
  FULLTEXT INDEX ft_content (content) WITH PARSER ngram
) ENGINE=InnoDB;

SHOW VARIABLES LIKE 'innodb_ft_min_token_size';  -- 기본 3
SHOW VARIABLES LIKE 'ngram_token_size';          -- 기본 2 (변경시 인덱스 재구축)

-- LIKE '%456%' 대체
SELECT * FROM posts WHERE MATCH(content) AGAINST('+456' IN BOOLEAN MODE);
```

## 5. 성능 팁·적용 판단

- WHERE 조건과 함께 사용, MATCH 칼럼 순서 = 인덱스 정의 순서, 대량 UPDATE 후 인덱스 재구성.
- **적합**: 단순 게시판·중소규모. **부적합**: 형태소 한글검색·자동완성·동의어·랭킹 → Elasticsearch 고려.

## 6. 연관 개념

- [[2026-06-14-MS09_(MySQL-Ngram-Full-Text-Search-장단점)]]
- [[2026-06-14-MS06_(MySQL-Functional-Index-FBI-LIKE-한계)]]
- [[2026-06-14-Elasticsearch-개요-허브]] — 형태소·랭킹 필요 시 대안
