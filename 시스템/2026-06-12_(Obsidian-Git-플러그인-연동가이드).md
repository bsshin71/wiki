# Obsidian Git 플러그인 설치 및 특정 폴더 연동 가이드
- **카테고리**: #시스템 #Obsidian #Git
- **태그**: #obsidian #git #github #backup #sync #windows
- **작성일**: 2026-06-12
- **참조 원본**: [[2026-06-12_Obsidian Git 플러그인 연동가이드]]

## 1. 핵심 요약
- Obsidian vault 전체가 아닌 **특정 하위 폴더(`wiki/`)만** GitHub Private 레포지토리와 선택적으로 동기화하는 설정 가이드.
- **Custom base path** 설정으로 vault 루트가 아닌 `wiki/` 폴더를 Git 루트로 지정.

## 2. 상세 설명

### 사전 준비

1. **GitHub Private 레포지토리 생성**
   - ⚠️ **Initialize with README** 반드시 체크 → `main` 브랜치 사전 생성
2. **PAT (Personal Access Token) 발급**
   - Settings → Developer Settings → Tokens (classic)
   - `repo` 권한 전체 체크, 토큰 문자열(`ghp_...`) 안전하게 보관

---

### 로컬 폴더 Git 초기화 및 첫 푸시 (PowerShell)

```powershell
# 연동할 하위 폴더로 이동
cd D:\llmwiki\wiki

# safe.directory 예외 등록 (소유권 보안 에러 방지)
git config --global --add safe.directory D:/llmwiki/wiki

# Git 초기화
git init

# 한글 파일명·줄바꿈 설정
git config --global core.quotepath false
git config --global core.autocrlf true

# 브랜치 설정 및 원격 연결
git branch -M main
git remote add origin https://github.com/bsshin71/wiki.git

# 원격 README 가져오기 (histories 불일치 허용)
git pull origin main --allow-unrelated-histories

# [충돌 발생 시] README.md 충돌 로컬 버전으로 해결
git checkout --ours README.md
git add README.md
git commit -m "Resolve merge conflict using local README"

# 첫 푸시
git push -u origin main
```

---

### Obsidian Git 플러그인 설치 및 설정

**설치:** Community Plugins → Obsidian Git 검색 → Install → Enable

**핵심 설정 항목:**

| 섹션 | 설정 항목 | 값 |
|------|-----------|-----|
| **Advanced** | Custom base path (Git repository path) | `wiki` |
| Automatic | Auto commit-and-sync interval | `15` 또는 `30` (분) |
| Automatic | Auto commit-and-sync after stopping file edits | ✅ 활성화 |
| Pull | Pull on startup | ✅ 활성화 |

> ⚠️ **설정 후 Obsidian 완전 재시작 필수.** 재시작해야 플러그인이 하위 폴더 Git 구조를 스캔하여 명령어 락이 해제됩니다.

---

### 동기화 테스트

```
Ctrl+P → "Git: Commit-and-sync" 실행
```

- 변경 없음: `No changes to commit`
- 변경 있음: 수정된 파일 개수만큼 push 및 동기화

---

### GitHub 렌더링 팁

마크다운 제목(`#`) 아래 반드시 **빈 줄** 삽입 → GitHub 웹 렌더링 정상:

```markdown
# 제목

본문 내용...
```

## 3. 연관 개념 (지식 연결)
- [[2026-06-05_(Claudian-GitHub-MCP-설정가이드)]] — GitHub PAT 발급·레포 연결 방식 동일 (Claude Code에서의 GitHub 연동)
- [[2026-06-12_(LLMWiki-Graphify-통합-설정-로그)]] — Claude Code의 GitHub 업로드 기능을 이 플러그인으로 대체한 설계 결정 포함
- [[2026-06-12_(LLMWiki-시스템-전체-가이드)]] — 전체 시스템 구조·프로세스 통합 가이드
