# claude-unity 플러그인

Unity 프로젝트를 위한 Claude Code CLI 전용 플러그인.

세션 관리 하네스 + 코드 리뷰 + 설계 플랜 + 문서화 + pre-commit 자동 리뷰를 제공합니다.

---

## 포함 컴포넌트

### Skills (슬래시 커맨드)

| 커맨드 | cost | 설명 |
|--------|------|------|
| `/setup` | sonnet | Unity 프로젝트 초기화 (최초 1회) |
| `/context-load` | haiku | 세션 시작 — 진행 상황 복구 |
| `/context-save` | haiku | 세션 종료 — 진행 상황 저장 + 커밋 |
| `/plan [기능명]` | opus | 설계 플랜 생성 (codebase-explorer → architect-planner) |
| `/review [파일]` | opus | 코드 리뷰 (codebase-explorer → unity-reviewer) |
| `/audit` | opus | 전체 감사 (코드 품질 + 빌드 체크 병렬 실행) |
| `/refactor [대상]` | opus | 리팩토링 플랜 + 단계별 실행 |
| `/debug [오류]` | sonnet | 버그 진단 + 수정 (debugger 에이전트) |
| `/analyze [대상]` | sonnet | 코드·시스템 분석 → docs/analysis/ 저장 |
| `/git-summary` | haiku | 커밋 단위 분리 + 메시지 제안 |
| `/doc readme\|handover\|delivery` | sonnet | 프로젝트 문서 생성 (doc-writer 에이전트) |

### Reference Skills (내부 자동 참조)

| 스킬 | 참조처 |
|------|--------|
| `unity-patterns` | architect-planner 에이전트 |
| `unity-review-rules` | unity-reviewer 에이전트 (references/checklist.md) |
| `architect-output` | architect-planner 에이전트 |
| `doc-templates` | doc-writer 에이전트 (references/templates.md) |

### Agents (스킬에서 위임 호출)

| 에이전트 | model | 호출처 |
|----------|-------|--------|
| `unity-reviewer` | sonnet | /review, /audit |
| `architect-planner` | opus | /plan, /refactor |
| `codebase-explorer` | haiku | /plan, /review, /refactor, /analyze |
| `debugger` | sonnet | /debug |
| `doc-writer` | sonnet | /doc |

### Hooks (자동 실행)

| 이벤트 | 동작 |
|--------|------|
| `SessionStart` | claude-progress.txt 또는 feature_list.json 자동 로드 |
| `PreToolUse (Write\|Edit *.cs)` | Unity C# 네이밍·코딩 규칙 컨텍스트 주입 |
| `pre-commit (git hook)` | .meta 확인 → 컴파일 → 코드 리뷰 → 문서 자동화 |

---

## 설치 방법

### Claude Code CLI

```bash
# 플러그인 설치
claude plugin install claude-unity.plugin

# Unity 프로젝트 루트에서 초기화 (최초 1회)
/setup
```

### pre-commit hook

`/setup` 실행 시 자동 설치됩니다. 수동 설치:
```bash
cp .claude/hooks/pre-commit .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

---

## 워크플로우

```
새 세션 시작
  → SessionStart hook 자동 실행 (진행 상황 로드)
  → /context-load               (수동 복구 시)

기능 개발
  → /plan [기능명]              (설계 플랜 → 승인 → 구현)
  → 코드 작성 (PreToolUse hook: C# 규칙 자동 주입)
  → /review [파일]              (코드 리뷰)
  → /audit                      (전체 감사)

세션 종료
  → /context-save               (진행 상황 저장 + 커밋)

git commit 시 자동 실행
  → pre-commit hook
     [0] .meta 누락 확인
     [1] Unity 컴파일 검증
     [2] 코드 리뷰 (Critical 있으면 커밋 차단)
     [3] 문서 자동 생성 → docs/analysis/
```

---

## 트리거 키워드 (자연어 자동 매칭)

| 말하면 | 실행되는 스킬 |
|--------|--------------|
| "에러", "버그", "NullReferenceException" | `/debug` |
| "리뷰해줘", "코드 검토", "문제 있어?" | `/review` |
| "설계해줘", "플랜 짜줘", "구조 잡아줘" | `/plan` |
| "리팩토링", "코드 정리", "개선해줘" | `/refactor` |
| "전체 감사", "배포 전 확인", "빌드 체크" | `/audit` |
| "분석해줘", "어떻게 동작해", "설명해줘" | `/analyze` |
| "문서 만들어줘", "인수인계", "납품 문서" | `/doc` |
| "커밋 메시지", "변경사항 정리" | `/git-summary` |
| "세션 시작", "이어서 작업" | `/context-load` |
| "오늘 마무리", "작업 저장" | `/context-save` |

---

## 요구 사항

- Claude Code CLI
- Unity 6 LTS (6000.0.x)
- Git Bash (Windows)
- Python 3 (hooks.json의 PreToolUse 훅 동작에 필요)
- `claude` CLI가 PATH에 등록 (pre-commit 리뷰 동작에 필요)

---

## 프로젝트 구조

```
claude-unity-plugin/
├── .claude-plugin/
│   └── plugin.json          ← 플러그인 메타데이터
├── CLAUDE.md                ← 플러그인 기본 컨텍스트
├── rules/                   ← 응답·Git·코드리뷰 규칙
├── engines/unity.md         ← Unity 6 스크립트 규칙
├── languages/csharp.md      ← C# 스타일 가이드
├── domains/unity.md         ← Unity 도메인 패턴
├── skills/                  ← 슬래시 커맨드 (11개)
├── agents/                  ← 위임 에이전트 (5개)
└── hooks/
    ├── hooks.json           ← Claude Code 훅 (SessionStart, PreToolUse)
    └── pre-commit           ← Git pre-commit 스크립트
```
