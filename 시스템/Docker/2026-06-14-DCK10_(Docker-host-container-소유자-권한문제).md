# Docker host·container 소유자 권한 문제 해결

- **카테고리**: #시스템 #Docker
- **태그**: #Docker #volume #permission #UID #GID #권한
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Docker host 와 container 소유자가 다른 권한문제 - BigDataTeam]]

## 1. 핵심 요약

호스트 디렉터리를 컨테이너에 bind mount할 때 **호스트 소유자 UID/GID와 컨테이너 실행 유저가 달라** Permission denied 오류가 발생한다. 컨테이너 실행 시 `user: "${UID}:${GID}"` 로 맞추거나, 호스트 디렉터리 권한을 조정한다.

---

## 2. 해결 방법 1 — UID/GID 맞추기 (권장)

```bash
# .env에 현재 사용자 UID/GID 저장
echo "UID=$(id -u)" > .env
echo "GID=$(id -g)" >> .env
```

```yaml
# docker-compose.yml
version: "3.9"
services:
  app:
    image: your-image
    volumes:
      - ./data:/app/data
    user: "${UID}:${GID}"   # 호스트 사용자와 동일 UID:GID로 실행
```

## 3. 해결 방법 2 — 호스트 권한 조정

```bash
# 컨테이너 내 실행 유저(예: UID 1000)로 소유자 변경
sudo chown -R 1000:1000 ./data

# 또는 전체 쓰기 권한 (보안상 비추천)
chmod -R 777 ./data
```

## 4. 연관 개념

- [[2026-06-14-DCK06_(Docker-볼륨-호스트-컨테이너-도커볼륨)]]
- [[2026-06-14-DCK05_(Docker-Compose-주요-명령)]]
