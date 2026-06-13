# Elasticsearch Rollover Alias 개념

- **카테고리**: #elasticsearch #인덱스구성
- **태그**: #elasticsearch #rollover #alias #시계열
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-rollover alias - BigDataTeam]]

## 1. 핵심 요약

Rollover Alias는 **하나의 별칭(alias)이 현재 write 인덱스를 가리키고**, 조건(크기·문서 수·기간) 충족 시 새 인덱스로 자동 전환(rollover)하는 시계열 데이터 관리 기능입니다.
클라이언트는 실제 인덱스명을 몰라도 **alias로 최신 인덱스에 일관되게 접근**합니다.

---

## 2. 동작 방식

1. **Alias 설정**: alias가 현재 write 인덱스를 가리킴(데이터 삽입 대상).
2. **Rollover**: 조건 충족 시 새 인덱스 생성 + alias가 새 인덱스로 이동. 기존 인덱스는 조회만 가능.
3. **조건**: 인덱스 크기/문서 수/시간 기반.

## 3. 설정 예시

```json
# 1) 초기 인덱스 + alias
PUT /logs-000001 { "settings": { "number_of_shards": 1 } }
POST /logs-000001/_alias/logs

# 2) rollover 조건 (예: 50GB 초과 시)
POST /logs/_rollover { "conditions": { "max_size": "50gb" } }
# → logs-000002 자동 생성, alias가 새 인덱스로 이동
```

> 조건 종류: `max_size`, `max_docs`, `max_age`.

## 4. 장점

- **자동화된 데이터 관리**: 새 인덱스 자동 생성으로 관리 간소화.
- **효율적 저장소**: 오래된 인덱스 분리, 최신 데이터 성능 유지.
- **편리한 조회**: alias만으로 최신 인덱스 쿼리(실제 인덱스명 불필요).

## 5. 연관 개념

- [[2026-06-14-ES10_(Elasticsearch-Hot-Warm-Delete-ISM-Rollover-구성)]]
- [[2026-06-14-ES13_(Elasticsearch-Index명-날짜함수-Date-Math)]]
- [[2026-06-14-ES06_(Elasticsearch-Index-관련-명령-매핑-Field삭제-Rollover)]]
