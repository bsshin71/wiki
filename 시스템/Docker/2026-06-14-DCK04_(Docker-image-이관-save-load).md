# Docker 이미지 이관 (save·load)

- **카테고리**: #시스템 #Docker
- **태그**: #Docker #image #save #load #이관 #offline
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-docker image 이관 - BigDataTeam]]

## 1. 핵심 요약

네트워크 단절 환경이나 레지스트리 없이 Docker 이미지를 이동할 때 `docker save`로 tar 파일로 저장하고 `docker load`로 로드한다.

---

## 2. 이미지 저장 (save)

```bash
docker save -o 저장할파일.tar REPOSITORY:TAG

# 복수 이미지 예시
docker save -o pgautoupgrade.tar pgautoupgrade/pgautoupgrade:latest
docker save -o redis.tar redis:7-alpine
docker save -o redash.tar redash/redash:10.1.0.b50633
docker save -o nginx.tar redash/nginx:latest
```

## 3. 이미지 로드 (load)

```bash
docker load -i 파일

docker load -i ./nginx.tar
docker load -i ./pgautoupgrade.tar
docker load -i ./redash.tar
docker load -i ./redis.tar

# 확인
docker images
```

## 4. 연관 개념

- [[2026-06-14-DCK01_(Docker-설치-offline-RHEL)]] — 오프라인 환경에서 이미지 이동
- [[2026-06-14-DCK07_(Docker-컨테이너-볼륨-백업복원)]]
