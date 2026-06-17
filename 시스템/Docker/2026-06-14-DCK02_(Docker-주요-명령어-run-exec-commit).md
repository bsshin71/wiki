# Docker 주요 명령어 (run·exec·commit)

- **카테고리**: #시스템 #Docker
- **태그**: #Docker #command #run #exec #commit
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Docker 주요 명령 - BigDataTeam]]

## 1. 핵심 요약

Docker 실무 핵심 명령: **`run`(컨테이너 생성·실행)**, **`exec`(실행 중 컨테이너 접속·명령)**, **`commit`(컨테이너→이미지 변환)**.

---

## 2. 주요 명령

```bash
# 컨테이너 실행 대기 (CMD 없거나 즉시 종료되는 이미지)
docker run -d -p 8080:8080 -p 6000:6000 --name alarm-server alarm-server sleep infinity

# 실행 중 컨테이너에서 명령 실행
docker exec -it alarm-server python main_adminserver.py
docker exec -it alarm-server /bin/sh      # 쉘 접속 (디버깅)

# 주요 관리 명령
docker ps                # 실행 중 컨테이너
docker ps -a             # 전체 컨테이너
docker stop <name>
docker rm <name>
docker rmi <image>
docker images            # 이미지 목록
docker logs -f <name>    # 로그 실시간
```

## 3. 연관 개념

- [[2026-06-14-DCK03_(Dockerfile-문법-build-run)]]
- [[2026-06-14-DCK05_(Docker-Compose-주요-명령)]]
