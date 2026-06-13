# Percona Server 8.0.40 패치과정

- **카테고리**: #DBMS #MySQL #Installation
- **태그**: #install #upgrade #patch #telemetry
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-percona server 8.0.40 패치과정 - BigDataTeam]]

## 1. 핵심 요약

Percona Server를 8.0.40으로 RPM 업그레이드(`rpm -Uvh ... --force`)하는 과정으로, **`libatomic` 의존성**과 **`percona-telemetry-agent` 의존성** 두 가지를 해결해야 합니다.
telemetry는 설치 후 비활성화하거나 `PERCONA_TELEMETRY_DISABLE=1`로 설치 시 끌 수 있습니다.

---

## 2. 패키지 업그레이드 (순서 주의)

```bash
systemctl stop mysql

rpm -Uvh percona-server-shared-compat-8.0.40-31.1.el8.x86_64.rpm --force
rpm -Uvh percona-server-shared-8.0.40-31.1.el8.x86_64.rpm --force
rpm -Uvh percona-server-devel-8.0.40-31.1.el8.x86_64.rpm --force
rpm -Uvh percona-server-client-8.0.40-31.1.el8.x86_64.rpm --force
rpm -Uvh percona-icu-data-files-8.0.40-31.1.el8.x86_64.rpm --force
rpm -Uvh percona-server-server-8.0.40-31.1.el8.x86_64.rpm --force
```

---

## 3. 의존성 오류 해결

### 3.1 libatomic 누락

```
error: Failed dependencies:
  libatomic.so.1()(64bit) is needed by percona-server-server-8.0.40
  percona-telemetry-agent is needed by percona-server-server-8.0.40
```

```bash
yum install libatomic
```

### 3.2 percona-telemetry-agent 누락

```bash
# Percona Distribution for MySQL 에서 다운로드 후 설치
rpm -ivh percona-telemetry-agent-1.0.1-1.el8.x86_64.rpm

# server 패키지 재설치
rpm -Uvh percona-server-server-8.0.40-31.1.el8.x86_64.rpm --force
```

> `Header V4 RSA/SHA512 Signature ... NOKEY` 경고는 GPG 키 미등록 경고로, `--force` 설치 시 무시 가능.

---

## 4. telemetry 비활성화

```bash
systemctl stop percona-telemetry-agent
systemctl disable percona-telemetry-agent
```

설치 단계에서 끄려면:

```bash
sudo PERCONA_TELEMETRY_DISABLE=1 yum install percona-server-server
```

---

## 5. 시작 및 버전 확인

```bash
systemctl start mysql
```

```sql
SELECT version();
-- 8.0.40-31
```

> 참고: https://docs.percona.com/percona-server/8.0/telemetry.html

---

## 6. 연관 개념

- [[2026-06-13-14_(MySQL-Percona-설치-상세-가이드)]]
- [[2026-06-13-39_(Percona-MySQL-8.4-vs-8.0-비교)]]
