# Docker 오프라인 설치 (RHEL/CentOS)

- **카테고리**: #시스템 #Docker
- **태그**: #Docker #install #offline #RHEL #yumdownloader
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-offline 으로 docker 설치하기 - BigDataTeam]]

## 1. 핵심 요약

인터넷이 안 되는 폐쇄망 서버에 Docker CE를 설치하려면 **인터넷 가능 서버에서 yumdownloader로 의존성 포함 RPM 다운로드 → scp 복사 → localinstall** 순서로 진행한다.

---

## 2. 절차 (RHEL 8.x)

```bash
# [인터넷 가능 서버] 패키지 다운로드
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
mkdir -p ~/docker-offline && cd ~/docker-offline
yumdownloader --resolve --destdir=. docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 파일 복사
scp ~/docker-offline/*.rpm user@offline-server:/tmp/docker-offline/

# [오프라인 서버] 설치
cd /tmp/docker-offline
sudo yum localinstall -y *.rpm
sudo systemctl enable --now docker
docker version
```

> 컨테이너 이미지는 `docker save`/`docker load`로 tar 파일 이동.

## 3. 연관 개념

- [[2026-06-14-DCK02_(Docker-주요-명령어-run-exec-commit)]]
- [[2026-06-14-DCK04_(Docker-image-이관-save-load)]]
