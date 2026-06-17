# PMM Client 설치 (운영 서버)

- **카테고리**: #DBMS #monitoring
- **태그**: #PMM #monitoring #pmm-client #MySQL #slow-query #percona
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-PMM 설치 (운영 서버) - BigDataTeam]]

## 1. 핵심 요약

각 DB 인스턴스에 **pmm-client**를 설치하고 PMM 서버에 등록하면 Grafana 대시보드에서 MySQL 메트릭·슬로우 쿼리를 모니터링할 수 있다. PMM은 slow query log 방식(Percona 권장)을 사용하며, MySQL 전용 모니터링 계정이 필요하다.

---

## 2. 설치 (RHEL/CentOS)

```bash
# Docker 설치
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker

# pmm-server 설치 (8090:80, 443:443)
sudo docker pull percona/pmm-server:2
sudo docker create --volume /srv --name pmm-data percona/pmm-server:2 /bin/true
sudo docker run --detach --restart always \
  --publish 8090:80 --publish 443:443 \
  --volumes-from pmm-data --name pmm-server percona/pmm-server:2

# pmm-client 설치
sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
sudo yum install -y pmm2-client
```

## 3. PMM 서버 등록

```bash
# 기본 등록
sudo pmm-admin config --server-insecure-tls \
  --server-url=https://user:password@10.11.6.56:443

# 클라이언트 IP 명시 (필요시)
sudo pmm-admin config --server-insecure-tls \
  --server-url=https://user:password@10.11.6.56:443 --force 10.11.14.143
```

## 4. MySQL 모니터링 계정 생성

```sql
CREATE USER 'pmm'@'localhost' IDENTIFIED BY 'password' WITH MAX_USER_CONNECTIONS 10;
GRANT SELECT, PROCESS, SUPER, REPLICATION CLIENT, RELOAD ON *.* TO 'pmm'@'localhost';
```

## 5. Slow Query 수집 설정 (my.cnf)

```ini
log_output=file
slow_query_log=ON
long_query_time=0
log_slow_rate_limit=100
log_slow_rate_type=query
log_slow_verbosity=full
innodb_monitor_enable=all
```

## 6. 연관 개념

- [[2026-06-14-PMM04_(PMM-모니터링-서버-구축-Docker)]]
- [[2026-06-14-PMM03_(PMM-AlertManager-설정)]]
