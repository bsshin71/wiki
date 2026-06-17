# Airflow 개발환경 구축 (VSCode Dev Container)

- **카테고리**: #airflow #install
- **태그**: #airflow #install #vscode #devcontainer #remote-ssh #dag
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-airflow 개발환경 구축 - BigDataTeam]]

## 1. 핵심 요약

DAG 개발 시 Type hint·Auto Completion이 안 되는 불편을 해결하기 위해, **Linux 서버에 Docker로 Airflow 설치 + VSCode Remote-SSH + Dev Containers** 조합으로 개발환경을 구성한다. Dockerfile 기반 컨테이너 내부에서 인터프리터가 자동 선택돼 패키지 자동완성이 동작한다.

---

## 2. 구성 단계

1. **VSCode 설치**
2. **Linux 서버에 Docker로 Airflow 설치** (→ [[2026-06-14-AF02_(Airflow-Docker-Compose-설치)]])
3. VSCode **Remote-SSH** 익스텐션 설치 → 원격 서버 접속
4. **Dev Containers** 익스텐션 설치
5. `CMD+Shift+P` → `Dev Containers: Open Folder in Container...` → `Dockerfile` 폴더 선택 → `From Dockerfile`
   - 해당 이미지로 컨테이너 생성, VSCode가 내부 인터프리터 자동 선택 → **type hint + auto completion** 제공

## 3. 효과

| 문제 | 해결 |
|------|------|
| Interpreter 미설정으로 type hint 없음 | Dev Container 내부 인터프리터 자동 사용 |
| 설치 패키지 자동완성 불가 | 컨테이너에 설치된 provider 패키지 인식 |

## 4. 연관 개념

- [[2026-06-14-AF02_(Airflow-Docker-Compose-설치)]]
- [[2026-06-14-AF04_(Airflow-기본-Usecase-SSHOperator-Telegram)]]
