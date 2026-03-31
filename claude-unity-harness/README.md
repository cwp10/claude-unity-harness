# claude-unity-harness 플러그인

Unity 프로젝트를 위한 Claude Code CLI 전용 플러그인.

세션 관리 하네스 + 코드 리뷰 + 설계 플랜 + 문서화 + pre-commit 자동 리뷰를 제공합니다.

---

## 설치 방법

### Claude Code CLI

```
/plugin marketplace add https://github.com/cwp10/claude-unity-harness
/plugin install claude-unity-harness
```

초기화 및 상태 확인:

```
/setup
/setup-check
```

### pre-commit hook

`/setup` 실행 시 자동 설치됩니다. 재설치가 필요하면:
```bash
/setup --force
```

---

## 포함 컴포넌트

### Skills (슬래시 커맨드)

| 커맨드 | cost | 설명 |
|--------|------|------|
| `/setup` | sonnet | Unity 프로젝트 초기화 (최초 1회) |
| `/setup-check` | haiku | 플러그인·hook·하네스 파일 설치 상태 진단 |
| `/deep-interview [기능명]` | sonnet | /plan 전 요구사항 명확화 |
| `/context-load` | haiku | 세션 시작 — project-memory.json + 진행 상황 복구 |
| `/context-save` | haiku | 세션 종료 — project-memory.json + 진행 상황 저장 + 커밋 |
| `/plan [기능명]` | opus | 설계 플랜 생성 (codebase-explorer → architect-planner) |
| `/review [파일]` | sonnet | 코드 리뷰 (codebase-explorer → unity-reviewer) |
| `/audit` | sonnet | 전체 감사 (코드 품질 + 빌드 체크 병렬 실행) |
| `/refactor [대상]` | opus | 리팩토링 플랜 + 단계별 실행 |
| `/debug [오류]` | sonnet | 버그 진단 + 수정 (debugger 에이전트) |
| `/analyze [대상]` | sonnet | 코드·시스템 분석 → docs/analysis/ 저장 |
| `/git-summary` | haiku | 커밋 단위 분리 + 메시지 제안 |
| `/doc readme\|handover\|delivery` | sonnet | 프로젝트 문서 생성 (doc-writer 에이전트) |

### Reference Skills (내부 자동 참조)

| 스킬 | 참조처 |
|------|--------|
| `unity-patterns` | architect-planner 에이전트, /plan 스킬 |

### Agents (스킬에서 위임 호출)

| 에이전트 | model | 역할 | 호출처 |
|----------|-------|------|--------|
| `unity-reviewer` | sonnet | Unity C# 코드 독립 리뷰 (Critical/Warning/Suggestion) | /review, /audit |
| `architect-planner` | opus | 설계 플랜 생성 + 승인 후 docs/architecture/ 저장 | /plan, /refactor |
| `codebase-explorer` | haiku | 코드베이스 구조 탐색 및 관련 파일 수집 | /plan, /review, /refactor, /analyze |
| `debugger` | sonnet | 버그 진단 + 수정 코드 제안 | /debug |
| `doc-writer` | sonnet | XML 주석·README·인수인계·납품 문서 생성 | /doc |
| `verifier` | sonnet | 기능 구현 완료 후 스펙 충족 검증 → passes:true | 기능 구현 완료 후 |
| `critic` | sonnet | 악마의 변호인 — GC/드로우콜/설계 관점 대안 제시 | /review 후, 최적화 요청 시 |

### Hooks (자동 실행)

| 이벤트 | 동작 |
|--------|------|
| `SessionStart` | project-memory.json → claude-progress.txt 순으로 자동 로드 |
| `UserPromptSubmit` | 첫 메시지 시 컨텍스트 로드 (SessionStart 폴백, 중복 방지) |
| `SessionEnd` | .context_loaded 플래그 제거 → 다음 세션 컨텍스트 로드 준비 |
| `PreToolUse (Write\|Edit *.cs)` | Unity C# 네이밍·코딩 규칙 컨텍스트 주입 |
| `PostToolUse (Write *.cs)` | 신규 .cs 파일 생성 시 .meta 누락 경고 |
| `pre-commit (git hook)` | .meta 확인 → 컴파일 → 코드 리뷰 → 문서 자동화 |

---

## 워크플로우

```
새 세션 시작
  → SessionStart hook 자동 실행 (project-memory.json 로드)
  → /context-load               (수동 복구 시)

기능 개발
  → /deep-interview [기능명]    (요구사항 불명확 시 — 선택)
  → /plan [기능명]              (설계 플랜 → 승인 → 구현)
  → 코드 작성 (PreToolUse hook: C# 규칙 자동 주입)
  →           (PostToolUse hook: .meta 누락 자동 경고)
  → /review [파일]              (코드 리뷰)
  → critic 에이전트             (대안·최적화 탐색 — 선택)
  → verifier 에이전트           (스펙 충족 검증 → passes: true)
  → /audit                      (전체 감사 — 배포 전)

세션 종료
  → /context-save               (project-memory.json + git 커밋)
  → SessionEnd hook 자동 실행  (.context_loaded 플래그 제거)

git commit 시 자동 실행
  → pre-commit hook
     [0] .meta 누락 확인
     [1] Unity 컴파일 검증
     [2] 코드 리뷰 (Critical 있으면 커밋 차단)
     [3] 문서 자동 생성 → docs/analysis/
```

---

## 트리거 키워드 (자연어 자동 매칭)

| 말하면 | 실행되는 스킬/에이전트 |
|--------|----------------------|
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
| "요구사항 정리", "뭘 만들어야 할지" | `/deep-interview` |
| "설치 확인", "환경 점검" | `/setup-check` |
| "더 나은 방법 없어?", "최적화 방법", "이게 최선이야?" | `critic 에이전트` |

---

## 프로젝트 메모리 구조 (.claude/project-memory.json)

`/context-save` 시 자동 생성·업데이트.

```json
{
  "techStack": "Unity 6 LTS, C# 9.0, URP",
  "platform": "Android/iOS",
  "currentFeature": "인벤토리 시스템",
  "phase": "구현 중",
  "conventions": {
    "naming": "m_(private) k_(const) s_(static)",
    "async": "UniTask",
    "assets": "Addressables"
  },
  "decisions": [
    { "date": "2026-03-31", "decision": "씬 관리에 Addressables 사용", "reason": "Resources.Load 성능 문제" }
  ],
  "blockers": [],
  "nextUp": "전투 시스템",
  "progress": "2/8 기능 완료",
  "lastUpdated": "2026-03-31"
}
```

---

## 요구 사항

- Claude Code CLI
- Unity 6 LTS (6000.0.x)
- Git Bash (Windows)
- `claude` CLI가 PATH에 등록 (pre-commit 리뷰 동작에 필요)

---

## 프로젝트 구조

```
claude-unity-harness/
├── .claude-plugin/
│   └── plugin.json          ← 플러그인 메타데이터
├── rules/                   ← 응답·Git·코드리뷰 규칙
├── engines/unity.md         ← Unity 6 스크립트 규칙
├── languages/csharp.md      ← C# 스타일 가이드
├── domains/unity.md         ← Unity 도메인 패턴
├── skills/                  ← 슬래시 커맨드 (13개)
│   └── unity-patterns/      ← Unity 패턴 Reference Skill
├── agents/                  ← 위임 에이전트 (7개)
│   ├── unity-reviewer.md
│   ├── architect-planner.md
│   ├── codebase-explorer.md
│   ├── debugger.md
│   ├── doc-writer.md
│   ├── verifier.md
│   └── critic.md
└── hooks/
    ├── hooks.json           ← Claude Code 훅 (SessionStart/End, UserPromptSubmit, PreToolUse, PostToolUse)
    └── pre-commit           ← Git pre-commit 스크립트
```
