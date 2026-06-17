# DB-ELK Filebeat — DB 로그 수집 설정

- **카테고리**: #elasticsearch #monitoring
- **태그**: #elasticsearch #monitoring #filebeat #DB로그 #mysql-log #log-collection
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-db-elk filebeat 구성 - BigDataTeam]]

## 1. 핵심 요약

Filebeat를 DB 서버에 설치해 **mysql-err.log·mysql-slow.log·시스템 로그**를 수집하고 Logstash(5044 포트)로 전송한다. `msg_type`·`env_type`·`src_type` 커스텀 필드를 추가해 ES에서 로그 유형별 필터링이 가능하다.

---

## 2. 설치

```bash
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.17.0-x86_64.rpm
yum localinstall filebeat-8.17.0-x86_64.rpm
systemctl enable filebeat
```

## 3. filebeat.yml 핵심 설정

```yaml
filebeat.inputs:
  # MySQL error log
  - type: log
    enabled: true
    paths: [/home/data/trc/mysql-err.log]
    fields:
      msg_type: myerr
      svc_name: bos
      env_type: prd
      src_type: database
    fields_under_root: true
    close_inactive: 5m
    close_renamed: true
    ignore_older: 48h
    clean_inactive: 96h
    scan_frequency: 2s

  # MySQL slow query log
  - type: log
    enabled: true
    paths: [/home/data/trc/mysql-query.log]
    fields: {msg_type: myslow, svc_name: bos, env_type: prd, src_type: database}
    fields_under_root: true
    ignore_older: 48h

output.logstash:
  hosts: ["logstash-server:5044"]
  ssl.certificate_authorities: ["/etc/filebeat/certs/ca.crt"]
```

> `ignore_older`: 48h 이상 변경 없는 파일은 무시. `clean_inactive` > `ignore_older` 설정 필요.

## 4. 연관 개념

- [[2026-06-14-DBELK04_(DB-ELK-Logstash-파이프라인-구성)]]
- [[2026-06-14-DBELK01_(DB-ELK-Docker-Compose-통합구성)]]
