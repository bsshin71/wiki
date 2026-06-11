# Claudian Smart Connections MCP 설정 가이드
- **카테고리**: #시스템 #Obsidian #MCP
- **태그**: #obsidian #claudian #mcp #smart-connections #windows
- **작성일**: 2026-06-05
- **참조 원본**: raw/2026-06-05_Claudian-Smart-Connections-MCP-설정가이드.md

## 1. 핵심 요약
- Obsidian + Claudian 환경에서 Smart Connections MCP를 연결하면 **로컬 벡터 임베딩** 기반 의미 검색으로 vault 탐색 토큰을 대폭 절감할 수 있다.
- Windows + nvm-windows 환경에서 **심볼릭 링크가 아닌 node.exe 실제 물리 경로** 사용이 핵심이다.

## 2. 상세 설명

### Smart Connections MCP를 쓰는 이유

| 구분 | 기존 방식 | Smart Connections MCP |
|------|----------|----------------------|
| 검색 방식 | grep/glob 텍스트 탐색 | 의미 기반 벡터 검색 |
| 키워드 일치 | 정확히 일치해야 찾음 | 비슷한 의미면 찾음 |
| 탐색 범위 | vault 전체 스캔 | 사전 인덱스 활용 |
| 토큰 소모 | 큼 | 작음 (관련 블록만 반환) |
| 비용 | - | 무료 (로컬 모델: TaylorAI/bge-micro-v2) |

**제공 MCP 도구:** `search_notes`, `get_similar_notes`, `get_connection_graph`, `get_note_content`

---

### 설치 절차 (8단계)

**사전 조건:** Node.js v20 LTS (v24 이상 사용 불가 — `ERR_UNSUPPORTED_ESM_URL_SCHEME` 오류 발생)

```powershell
# nvm-windows 사용 시
nvm install 20
nvm use 20
nvm current   # v20.x.x 확인
```

**STEP 1 — Obsidian Smart Connections 플러그인 설치**
- Settings → Community plugins → Smart Connections 설치/활성화
- 임베딩 완료 확인: `D:\llmwiki\.smart-env\multi\` 에 `.ajson` 파일 생성
- ⚠️ 임베딩 완료 전 다음 단계 진행 금지

**STEP 2~3 — MCP 서버 클론 및 빌드**

```powershell
mkdir C:\mcp-servers
cd C:\mcp-servers
git clone https://github.com/msdanyg/smart-connections-mcp.git smart-connections-mcp2
cd smart-connections-mcp2
npm install
npm run build
dir dist\index.js   # 파일 존재 확인
```

> ⚠️ 패키지 선택 주의:
> - `@yejianye/smart-connections-mcp` → Windows ESM 경로 버그 ❌
> - `dan6684/smart-connections-mcp` → Python 기반, numpy 빌드 실패 ❌
> - **`msdanyg/smart-connections-mcp`** → Windows 정상 동작 ✅

**STEP 4 — 수동 실행 테스트**

```powershell
$env:SMART_VAULT_PATH = "D:\llmwiki"
node C:\mcp-servers\smart-connections-mcp2\dist\index.js
# "Smart Connections MCP Server running on stdio" 확인 후 Ctrl+C
```

**STEP 5 — node.exe 실제 물리 경로 확인 ★ 가장 중요**

nvm-windows의 `C:\nvm4w\nodejs\node.exe`는 **심볼릭 링크**. Claudian stdio 실행 시 심볼릭 링크 해석 실패 → `MCP error -32000: Connection closed` 발생.

```powershell
# 심볼릭 링크 실제 타겟 확인
Get-Item "C:\nvm4w\nodejs" | Format-List FullName, Target, LinkType
# Target: C:\Users\bsshi\AppData\Local\nvm\v20.20.2

# 실제 물리 경로로 실행 검증
Test-Path "C:\Users\bsshi\AppData\Local\nvm\v20.20.2\node.exe"  # True 확인
```

**STEP 6 — mcp.json 직접 편집** (`D:\llmwiki\.claude\mcp.json`)

> ⚠️ Claudian UI Command 필드로는 안 됨 (Windows 경로 파싱 오류). JSON 파일 직접 편집 필수.

```json
{
  "mcpServers": {
    "smart-connection-mcp": {
      "command": "C:\\Users\\bsshi\\AppData\\Local\\nvm\\v20.20.2\\node.exe",
      "args": ["C:\\mcp-servers\\smart-connections-mcp2\\dist\\index.js"],
      "env": { "SMART_VAULT_PATH": "D:\\llmwiki" }
    }
  }
}
```

**STEP 7~8 — Obsidian 완전 재시작 → Verify**
- Settings → MCP Servers → smart-connection-mcp → Verify → ✅ Connection successful

---

### 실패 시 디버깅

| 증상 | 원인 | 해결 |
|------|------|------|
| `ERR_UNSUPPORTED_ESM_URL_SCHEME` | Node.js v24 사용 | v20 LTS 다운그레이드 |
| `Connection closed` (-32000) | nvm 심볼릭 링크 경로 | 실제 물리 경로 사용 (STEP 5) |
| `Module not found` | npm install 누락 | `npm install && npm run build` 재실행 |
| `Vault not found` | 환경변수 이름 오타 | `SMART_VAULT_PATH` 정확히 |
| `Loaded 0 notes` | 임베딩 미완료 | Smart Connections 플러그인 임베딩 확인 |

개발자 도구 확인: Obsidian `Ctrl+Shift+I` → Console 탭 → Verify 클릭 후 오류 메시지 확인

---

### .gitignore 권장 설정

```gitignore
.smart-env/   # Smart Connections 임베딩 (자동 생성)
.claudian/    # Claudian 내부 파일
.claude/      # MCP 설정 (민감정보 포함 가능)
```

---

### 핵심 교훈 요약

| 함정 | 해결 |
|------|------|
| Node.js v24 사용 | v20 LTS |
| `@yejianye` 또는 Python 기반 MCP 패키지 | `msdanyg/smart-connections-mcp` |
| Claudian UI Command 필드 사용 | `mcp.json` 직접 편집 |
| nvm 심볼릭 링크 경로 | 실제 물리 경로 (`AppData\Local\nvm\v20.20.2\`) |
| 환경변수 이름 추측 | `SMART_VAULT_PATH` 정확히 |

## 3. 연관 개념 (지식 연결)
- [[2026-06-05_(Claudian-GitHub-MCP-설정가이드)]] — 동일 패턴으로 GitHub MCP 설정 (node.exe 물리 경로, mcp.json 편집)
