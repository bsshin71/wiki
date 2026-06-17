# Dockerfile 문법 및 빌드·실행

- **카테고리**: #시스템 #Docker
- **태그**: #Docker #Dockerfile #build #run #CMD #python
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Dockerfile 문법 - BigDataTeam]]

## 1. 핵심 요약

Dockerfile로 이미지를 빌드하고 컨테이너를 실행하는 패턴. `CMD`는 **대기 상태로 종료되지 않는 명령**이어야 컨테이너가 계속 실행된다.

---

## 2. Dockerfile (Python 앱 예시)

```dockerfile
FROM python:3.12-slim
WORKDIR /alarm
COPY . /alarm
RUN pip3 install --upgrade pip
RUN pip3 install -r require4unix.txt
ENV ALARM_HOME /alarm/alarm
WORKDIR /alarm/alarm
CMD ["/bin/sh", "./script/run_all.sh"]   # 종료하지 않는 명령 필요
```

## 3. 빌드 스크립트 패턴

```bash
image="alarm-server"
container="alarm-server"

echo "$dockerfile" > Dockerfile
docker stop $container
docker rm $container
docker rmi $image
docker build --force-rm -t $image .
```

## 4. 컨테이너 실행

```bash
docker run -d \
  -p 8080:8080 -p 6000:6000 \
  --name alarm-server \
  --restart always \
  -e ALARM_HOME=/alarm/alarm \
  -v $PWD/conf:/alarm/alarm/conf \
  -v $PWD/logs:/alarm/alarm/logs \
  alarm-server
```

## 5. 연관 개념

- [[2026-06-14-DCK02_(Docker-주요-명령어-run-exec-commit)]]
- [[2026-06-14-DCK05_(Docker-Compose-주요-명령)]]
