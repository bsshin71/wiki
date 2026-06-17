# MySQL Innotop — InnoDB 특화 CLI 모니터링 도구

- **카테고리**: #DBMS #MySQL #tools
- **태그**: #MySQL #tools #innotop #monitoring #InnoDB #CLI #복제모니터링
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Innotop - MySQL용 CLI 기반 탑과 유사한 모니터 도구]]

## 1. 핵심 요약

`innotop`은 InnoDB 엔진 중심의 MySQL 실시간 모니터링 도구로, mytop보다 **InnoDB Buffer·I/O·Row 작업·Lock·복제 상태**까지 상세히 볼 수 있다. Perl 기반이며 멀티모드로 전환 가능하다.

---

## 2. 설치

```bash
# 패키지 관리자
sudo yum install innotop           # RHEL/CentOS/Fedora
sudo apt install innotop           # Debian/Ubuntu

# 소스 설치 (패키지 미제공 시)
git clone https://github.com/innotop/innotop.git && cd innotop
sudo apt install cpanminus
cpanm Term::ReadKey DBI DBD::mysql
perl innotop
```

## 3. 접속·실행

```bash
innotop -u root -p 'your_password'
```

## 4. 주요 모드 (키 전환)

| 모드 | 내용 |
|------|------|
| 쿼리 목록 | `SHOW FULL PROCESSLIST` 출력 (mytop와 유사) |
| **InnoDB I/O** | 보류 I/O, 스레드, 파일 I/O, 로그 통계 |
| **InnoDB 버퍼** | Buffer Pool, 페이지 통계, Insert Buffer, AHI |
| **InnoDB 행 작업** | INSERT/UPDATE/READ/DELETE 건수 실시간 |
| 명령 요약 | 서버 명령 실행 횟수 |
| 변수·상태 | QPS, 연결 수, 캐시 사용량 등 |
| **복제 상태** | Slave Lag, IO/SQL Thread 상태 |

## 5. 연관 개념

- [[2026-06-14-MS28_(MySQL-mytop-CLI-모니터링-도구)]] — mytop과 비교
- [[2026-06-14-MS01_(MySQL-InnoDB-Adaptive-Hash-Index-AHI)]]
