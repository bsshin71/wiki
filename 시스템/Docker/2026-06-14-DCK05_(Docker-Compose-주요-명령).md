# Docker Compose 주요 명령

- **카테고리**: #시스템 #Docker
- **태그**: #Docker #compose #multi-container #yaml #env
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Docker compose - BigDataTeam]]

## 1. 핵심 요약

Docker Compose는 **여러 컨테이너를 YAML 파일 하나로 정의·실행·관리**하는 도구. 포트·네트워크·볼륨 설정을 한 곳에 모아 단일 명령으로 전체 스택을 구동하거나 종료한다.

---

## 2. 주요 명령

```bash
docker compose up -d          # 이미지 빌드 + 네트워크·볼륨 생성 + 컨테이너 실행
docker compose down           # 컨테이너·네트워크 삭제
docker compose down -v        # 볼륨도 함께 삭제
docker compose start          # 컨테이너 시작(기존 설정 유지)
docker compose stop           # 컨테이너 중지
docker compose logs -f        # 실시간 로그
docker compose ps             # 실행 중 서비스 목록
docker compose build          # 이미지 빌드만
docker compose pull           # 이미지 pull
```

## 3. .env 파일 활용

```ini
# .env
STACK_VERSION=8.17.0
ES_PORT=9200
KIBANA_PORT=5601
```
- `docker compose up` 시 `.env` 자동 로드 → `${STACK_VERSION}` 변수 치환
- 환경별(dev/prod) `.env.dev`·`.env.prod` 분리 운용 가능

## 4. 연관 개념

- [[2026-06-14-DCK03_(Dockerfile-문법-build-run)]]
- [[2026-06-14-AF02_(Airflow-Docker-Compose-설치)]] — Airflow Compose 예시
- [[2026-06-14-DBELK01_(DB-ELK-Docker-Compose-통합구성)]] — ELK Compose 예시
