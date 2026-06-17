# MySQL CLI — auto-rehash·탭 완성으로 인한 프로시저 오류

- **카테고리**: #DBMS #MySQL #tools
- **태그**: #MySQL #tools #mysql-cli #auto-rehash #tab-completion #stored-procedure
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-mysql cli - BigDataTeam]]

## 1. 핵심 요약

mysql CLI에서 탭 문자가 포함된 프로시저 DDL을 붙여넣으면 **auto-rehash(탭 자동완성)** 가 발동해 구문이 깨진다. `--skip-auto-rehash` 옵션으로 비활성화하고, 에디터에서 **탭을 스페이스로 변환** 후 붙여넣으면 해결된다.

---

## 2. 현상

```sql
mysql> DELIMITER ;;
mysql> CREATE DEFINER=`ces`@`%` PROCEDURE `SP_CHECK_...`(
Display all 797 possibilities? (y or n)   ← 탭 완성이 트리거됨
```
→ auto-rehash 활성 상태에서 붙여넣기 텍스트에 탭이 있으면 탭 완성이 실행되어 입력이 깨짐.

## 3. 해결

```bash
# 방법 1: auto-rehash 비활성화 옵션으로 접속
mysql --skip-auto-rehash -uroot -p
mysql --disable-auto-rehash -uroot -p
mysql --no-auto-rehash -uroot -p

# 방법 2: 텍스트 에디터에서 탭→스페이스 변환 후 붙여넣기
# VS Code: Ctrl+H → \t → (space×4), 정규식 모드

# 방법 3: SQL Client Tool(DBeaver, MySQL Workbench 등) 활용
```

## 4. 연관 개념

- [[2026-06-14-MS26_(MySQL-mysql_config_editor-login-path)]] — CLI 접속 보안
- [[2026-06-14-MS31_(MySQL-CLI-prompt-커스텀)]]
