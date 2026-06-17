# PMM 모니터링 서버 구축 (Docker)

- **카테고리**: #DBMS #monitoring
- **태그**: #PMM #monitoring #docker #percona #grafana #alertmanager
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-pmm 모니터링 서버 구축 - BigDataTeam]]

## 1. 핵심 요약

PMM(Percona Monitoring and Management) 서버를 **Docker**로 설치하고, 별도 AlertManager를 연동하는 절차. PMM 서버는 Grafana·Prometheus·Alertmanager를 포함한 All-in-One 컨테이너로 443 포트에 노출된다.

---

## 2. PMM Server 설치

```bash
# 이미지 pull
docker pull percona/pmm-server:2

# 데이터 볼륨 컨테이너 생성
docker create --volume /srv --name pmm-data percona/pmm-server:2 /bin/true

# PMM 서버 실행 (443 포트)
docker run --detach --restart always \
  --publish 443:443 \
  --volumes-from pmm-data \
  --name pmm-server percona/pmm-server:2

# 관리자 비밀번호 설정
docker exec -t pmm-server change-admin-password <new_password>
```

## 3. AlertManager 별도 구축 (PMM AlertManager와 연동)

```bash
# prometheus.io/download에서 AlertManager 다운로드
# alertmanager.yml 작성 후 실행
./alertmanager --config.file=alertmanager.yml
```

## 4. 연관 개념

- [[2026-06-14-PMM05_(PMM-Client-설치-운영서버)]] — pmm-client 설치
- [[2026-06-14-PMM03_(PMM-AlertManager-설정)]]
