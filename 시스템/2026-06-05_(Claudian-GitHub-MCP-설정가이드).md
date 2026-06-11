# Claudian GitHub MCP 설정 가이드
- **카테고리**: #시스템 #Obsidian #MCP
- **태그**: #obsidian #claudian #mcp #github #windows
- **작성일**: 2026-06-05
- **참조 원본**: raw/2026-06-05_Claudian-GitHub-MCP-설정가이드.md

## 1. 핵심 요약
- Obsidian + Claudian 환경에서 GitHub MCP를 연결하면 채팅에서 자연어로 vault 문서를 GitHub 레포지토리에 업로드할 수 있다.
- [[2026-06-05_(Claudian-Smart-Connections-MCP-설정가이드)]]와 동일한 패턴: **node.exe 실제 물리 경로 + mcp.json 직접 편집**.

## 2. 상세 설명

### GitHub MCP를 쓰는 이유

| 기능 | 설명 |
|------|------|
| 파일 업로드 | vault 문서를 레포지토리에 commit |
| 레포 탐색 | 파일 목록, 내용 조회 |
| 이슈/PR 관리 | 이슈 생성, PR 조회 |
| 브랜치 관리 | 브랜치 생성·전환 |

```
# Claudian 채팅에서 자연어로 업로드
@github 이 문서를 bsshin71/wiki 레포지토리에 업로드해줘
```

---

### 설치 절차 (8단계)

**사전 조건:** [[2026-06-05_(Claudian-Smart-Connections-MCP-설정가이드)]] 의 Node.js v20 환경과 동일.

**STEP 1 — GitHub Personal Access Token 발급**

1. https://github.com/settings/tokens → **Generate new token (classic)**
2. Scopes: **`repo` 전체 체크** (private 레포 접근 포함)
3. 토큰 문자열 메모장에 임시 저장 (다시 볼 수 없음)

> ⚠️ 토큰을 GitHub에 커밋하면 자동 무효화됨. `.claude/` 를 `.gitignore`에 반드시 추가.

**STEP 2 — 대상 레포지토리 준비**

```
owner: bsshin71
repo:  wiki
branch: main
```

**STEP 3 — npx-cli.js 절대 경로 확인**

```powershell
Get-ChildItem -Path "C:\Users\bsshi\AppData\Local\nvm\v20.20.2" -Filter "npx-cli.js" -Recurse
# 일반 경로: C:\Users\bsshi\AppData\Local\nvm\v20.20.2\node_modules\npm\bin\npx-cli.js

Test-Path "C:\Users\bsshi\AppData\Local\nvm\v20.20.2\node_modules\npm\bin\npx-cli.js"
# True 확인
```

**STEP 4 — 수동 실행 테스트**

```powershell
$env:GITHUB_PERSONAL_ACCESS_TOKEN = "ghp_여기에_발급받은_토큰"
& "C:\Users\bsshi\AppData\Local\nvm\v20.20.2\node.exe" `
  "C:\Users\bsshi\AppData\Local\nvm\v20.20.2\node_modules\npm\bin\npx-cli.js" `
  -y "@modelcontextprotocol/server-github"
# 서버 대기 상태 확인 후 Ctrl+C
```

**STEP 5 — mcp.json에 GitHub 항목 추가** (`D:\llmwiki\.claude\mcp.json`)

> Smart Connections가 이미 설정되어 있다면 그 설정은 유지하고 github 항목만 추가.

```json
{
  "mcpServers": {
    "smart-connection-mcp": {
      "command": "C:\\Users\\bsshi\\AppData\\Local\\nvm\\v20.20.2\\node.exe",
      "args": ["C:\\mcp-servers\\smart-connections-mcp2\\dist\\index.js"],
      "env": { "SMART_VAULT_PATH": "D:\\llmwiki" }
    },
    "github": {
      "command": "C:\\Users\\bsshi\\AppData\\Local\\nvm\\v20.20.2\\node.exe",
      "args": [
        "C:\\Users\\bsshi\\AppData\\Local\\nvm\\v20.20.2\\node_modules\\npm\\bin\\npx-cli.js",
        "-y",
        "@modelcontextprotocol/server-github"
      ],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_여기에_토큰" }
    }
  }
}
```

**주의사항:**
- `command` 는 `npx`가 아닌 **node.exe** (npx도 결국 node로 실행됨)
- 두 항목 사이에 **쉼표** 필수
- 모든 역슬래시 `\\` 두 개

**STEP 6~7 — Obsidian 완전 재시작 → Verify**
- Settings → MCP Servers → github → Verify → ✅ Connection successful

**STEP 8 — 동작 테스트**

```
@github 내 GitHub 레포지토리 목록을 보여줘
```
`bsshin71/wiki` 가 목록에 나오면 정상.

---

### 실패 시 디버깅

| 증상 | 원인 | 해결 |
|------|------|------|
| `Connection closed` (-32000) | node.exe 경로 오류 | 실제 물리 경로 사용 |
| `npx-cli.js not found` | 잘못된 npm 경로 | STEP 3으로 정확한 경로 재확인 |
| `Authentication failed` | 토큰 오류 | GitHub 토큰 재발급 |
| `Permission denied` | 토큰 권한 부족 | `repo` 권한 체크 확인 |
| `Resource not accessible` | 권한 누락 | 토큰에 `repo` 전체 권한 추가 |

---

### 보안 주의사항

- `mcp.json` 에는 GitHub PAT가 **평문** 저장됨 → **절대 GitHub에 커밋 금지**
- GitHub의 secret scanning이 자동으로 토큰 무효화
- `.gitignore` 에 `.claude/` 추가 필수
- 위키 업로드만 필요하다면 **Fine-grained PAT** 로 특정 레포만 접근 제한 권장

토큰 노출 시: https://github.com/settings/tokens 에서 즉시 Revoke → 신규 발급

---

### 핵심 교훈 요약

| 함정 | 해결 |
|------|------|
| Claudian UI Command 필드에 `npx -y ...` 직접 입력 | `mcp.json` 직접 편집 |
| `command: "npx"` 사용 | `command: "node.exe"` + npx-cli.js 절대 경로 |
| nvm 심볼릭 링크 경로 | 실제 물리 경로 사용 |
| `repo` 권한 누락 토큰 | 발급 시 `repo` 전체 체크 |
| `.claude/` 폴더를 git에 커밋 | `.gitignore` 에 추가 |

## 3. 연관 개념 (지식 연결)
- [[2026-06-05_(Claudian-Smart-Connections-MCP-설정가이드)]] — 동일 환경·동일 패턴의 Smart Connections MCP 설정 (선행 가이드)
- [[2026-06-12_(LLMWiki-Graphify-통합-설정-로그)]] — graphify 연동 설정 (GitHub 업로드를 Obsidian Git으로 대체한 결정 포함)
