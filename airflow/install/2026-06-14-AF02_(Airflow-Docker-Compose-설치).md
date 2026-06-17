# Airflow Docker Compose 설치

- **카테고리**: #airflow #install
- **태그**: #airflow #install #docker #compose #CeleryExecutor #provider
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-airflow 설치 (docker) - BigDataTeam]]

## 1. 핵심 요약

보안에 취약한 SQLite 대신 **PostgreSQL + Redis + CeleryExecutor** 구성을 공식 `docker-compose.yaml`(apache/airflow:2.5.0)로 띄운다. `AIRFLOW_UID` 설정 → `docker compose up airflow-init`(DB 초기화) → `docker compose up -d`. MySQL/PostgreSQL/SSH/Telegram 연동은 **provider 패키지**를 설치(또는 이미지 extend)해야 한다.

> ⚠️ 최소 사양: **메모리 4GB**.

---

## 2. Docker Engine 설치 (RHEL)

```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl start docker
sudo usermod -aG docker $USER          # 재로그인 후 적용
sudo systemctl enable docker.service containerd.service   # 부팅 자동실행
```

## 3. Airflow Compose 배포

```bash
mkdir airflow && cd airflow
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.5.0/docker-compose.yaml'
mkdir -p ./dags ./logs ./plugins ./config
echo -e "AIRFLOW_UID=$(id -u)" > .env     # root 소유 방지
docker compose up airflow-init             # DB 초기화 + 관리자 계정
docker compose up -d
```

구성 서비스: `postgres:13`, `redis`, `airflow-webserver(8080)`, `airflow-scheduler`, `airflow-worker(celery)`, `airflow-triggerer`, `airflow-init`, (옵션)`flower(5555)`.

## 4. airflow CLI 래퍼

```bash
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.5.0/airflow.sh'
chmod +x airflow.sh
# docker-compose → docker compose 로 수정 후 사용
./airflow.sh users create -e "x@x.com" -f admin -l admin -r Admin -u sypark
./airflow.sh config list
./airflow.sh dags list
```

## 5. Provider 설치 (MySQL·PG·SSH·Telegram)

```bash
# 방법 A: 컨테이너 접속 후 pip
docker exec -it airflow-airflow-webserver-1 /bin/bash
pip install apache-airflow-providers-{telegram,ssh,postgres,mysql}
```
```dockerfile
# 방법 B(권장): 이미지 extend 후 rebuild
FROM apache/airflow:2.10.3
COPY requirements.txt /
RUN pip install --no-cache-dir -r /requirements.txt
# docker-compose.yml: image: ${AIRFLOW_IMAGE_NAME:-extend_airflow:latest}
# docker build . --tag extend_airflow:latest && docker compose up -d
```

## 6. 컨테이너 내부 airflow.cfg 수정

```bash
docker exec -it -u root airflow-airflow-webserver-1 /bin/bash
apt update && apt install vim
vi ./airflow.cfg
```

## 7. 연관 개념

- [[2026-06-14-AF01_(Airflow-설치-pip-Metadb)]]
- [[2026-06-14-AF03_(Airflow-개발환경-VSCode-DevContainer)]]
- [[2026-06-14-AF05_(Airflow-동시실행-제어-parallelism)]]
