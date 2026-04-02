---
name: git-summary
description: >
  Git 변경사항을 분석해 논리적 커밋 단위와 메시지를 제안한다.
  커밋 전 MCP 연결 시 컴파일 에러를 자동 확인하고, 에러가 있으면 커밋을 차단한다.
  세션 종료·진행 상황 저장은 /context-save 사용.
  이 스킬은 커밋 메시지 작성과 단위 분리만 담당한다.
tools: Bash, Read, mcp__UnityMCP__read_console
model: haiku
triggers:
  - "커밋 메시지"
  - "오늘 작업 정리"
  - "git 요약"
  - "변경사항 정리"
  - "커밋 단위로 나눠줘"
  - "커밋해줘"
keywords: [git, commit, summary, changelog, diff]
---

다음 순서로 실행한다:

## 0단계: git 루트 확인 (필수)

```bash
git rev-parse --show-toplevel   # git 루트 절대 경로
git rev-parse --show-prefix     # 현재 위치가 루트에서 얼마나 하위인지 확인
```

**현재 위치가 git 루트와 다를 경우 (show-prefix 결과가 비어있지 않으면):**
- 이후 모든 `git add`, `git commit` 명령은 반드시 git 루트에서 실행한다
- 파일 경로는 git 루트 기준 상대경로를 사용한다

> 예시: 현재 위치 `/Projects/my-game/my-module/`
> git 루트 `/Projects/my-game/`
> git add 시: `cd /Projects/my-game && git add my-module/Assets/...`

---

## 1단계: 변경사항 분석

1. `git status` 로 현재 변경 파일 목록 확인
2. `git diff --stat` 로 변경 규모 파악
3. `git log --oneline -10` 로 최근 커밋 흐름 파악
4. 변경된 파일들을 Read로 읽어 내용 파악

---

## 2단계: 컴파일 에러 사전 확인

변경 파일 중 `.cs` 파일이 1개 이상 있으면 반드시 이 단계를 실행한다.

**MCP 연결 확인:**

`mcp__UnityMCP__read_console` 호출 시도:

### MCP 연결됨 → 컴파일 에러 자동 확인

`read_console` 결과에서 `error` 레벨 로그 필터링:

**에러 없음:**
```
✅ 컴파일 에러 없음 — 커밋 진행합니다.
```
→ 3단계로 진행

**에러 있음:**
```
🚫 컴파일 에러 감지 — 커밋을 차단합니다.

발견된 에러:
- [파일명]:[줄번호] [에러 메시지]
- ...

커밋 전 에러를 수정하세요.
수정 후 /git-summary 를 다시 실행하거나,
긴급 커밋이 필요하면 git commit --no-verify 를 사용하세요.
```
→ 여기서 종료. 커밋 진행하지 않는다.

### MCP 미연결 → 수동 확인 요청

```
⚠️  Unity MCP 미연결 — 컴파일 에러를 자동으로 확인할 수 없습니다.

Unity Editor 콘솔에서 에러(빨간 항목)가 없는지 직접 확인 후 계속하세요.
[확인했습니다 / 취소]
```
→ 사용자가 확인하면 3단계로 진행, 취소하면 종료.

---

## 3단계: 커밋 단위 제안

다음 형식으로 출력한다:

---
## 오늘 작업 요약

### 변경 파일
- 파일명: 변경 내용 한 줄 요약

### 논리적 커밋 단위 제안
**커밋 1**
```
[타입] 설명
```
파일: 파일1, 파일2

**커밋 2** (있을 경우)
```
[타입] 설명
```
파일: 파일3

### 주의사항
- 민감정보 포함 여부 (API 키, 비밀번호, .env 파일)
- Unity 프로젝트: .meta 파일 누락 여부
- 커밋 전 확인 필요 사항
---

마지막으로 "이 단위로 커밋할까요?" 라고 확인한다.
