# claude-unity-harness

Unity 프로젝트를 위한 Claude Code 플러그인.

세션 관리 하네스, 코드 리뷰, 설계 플랜, 문서화, pre-commit 자동 리뷰를 제공합니다.

---

## 설치

```
/plugin marketplace add https://github.com/cwp10/claude-unity-harness
/plugin install claude-unity-harness@cwp10-plugins
```

설치 후 Unity 프로젝트 루트에서 최초 1회 실행:

```
/setup
```

---

## 슬래시 커맨드

| 커맨드 | 설명 |
|--------|------|
| `/setup` | 프로젝트 초기화 — CLAUDE.md 스택 설정, 하네스 파일 생성, hook 설치 · [↓](#1-setup--프로젝트-초기화) |
| `/context-load` | 세션 시작 — 이전 진행 상황 복구 · [↓](#2-context-load--세션-복구) |
| `/context-save` | 세션 종료 — 진행 상황 저장 + git 커밋 · [↓](#3-context-save--세션-저장) |
| `/deep-interview [기능명]` | 요구사항 심층 인터뷰 → /plan 자동 연결 · [↓](#4-deep-interview-기능명--요구사항-명확화) |
| `/plan [기능명]` | 설계 플랜 생성 (Plan Mode 강제) · [↓](#5-plan-기능명--설계-플랜-생성) |
| `/review [파일]` | 코드 리뷰 · [↓](#6-review-파일--코드-리뷰) |
| `/audit` | 전체 감사 + 빌드 체크 · `migrate` `localize` 서브커맨드 포함 · [↓](#7-audit--전체-감사) |
| `/refactor [대상]` | 리팩토링 플랜 + 실행 (Plan Mode 강제) · [↓](#8-refactor-대상--리팩토링) |
| `/fix [오류]` | 버그 진단 + 수정 · [↓](#9-fix-오류--버그-수정) |
| `/analyze [대상]` | 코드·시스템 분석 · [↓](#10-analyze-대상--코드-분석) |
| `/git-summary` | 커밋 메시지 제안 · [↓](#11-git-summary--커밋-메시지-제안) |
| `/doc readme\|handover\|delivery` | 문서 생성 · [↓](#12-doc-readmehandoverdelivery--문서-생성) |
| `/setup-check` | 설치 상태 점검 · [↓](#13-setup-check--설치-상태-점검) |
| `/unity-setup-mcp` | Unity MCP 서버 연동 설정 · [↓](#14-unity-setup-mcp--unity-mcp-연동) |
| `/unity-console` | Unity 콘솔 에러·경고 분석 (UnityMCP 필요) · [↓](#15-unity-console--콘솔-분석-unitymcp-필요) |
| `/unity-test [EditMode\|PlayMode\|All]` | 테스트 실행 + 실패 원인 분석 (UnityMCP 필요) · [↓](#16-unity-test-editmodeplaymodeall--테스트-실행-unitymcp-필요) |
| `/unity-scene-audit` | 씬 품질 감사 — Missing·네이밍·성능 점검 (UnityMCP 필요) · [↓](#17-unity-scene-audit--씬-감사-unitymcp-필요) |
| `/perf [파일]` | 성능 패턴 탐지 — GC Alloc·캐싱 누락·Draw Call · [↓](#18-perf-파일--성능-분석) |
| `/allow [on\|off]` | 권한 자동 승인 토글 — git push는 항상 차단 · [↓](#19-allow-onoff--오토파일럿) |
| `/ship [기능명]` | 전체 개발 파이프라인 — 설계 승인 1회 후 구현→리뷰→감사→검증 자동 · [↓](#20-ship-기능명--전체-파이프라인) |
| `/ensure [대상]` | 검증 + 실패 시 자동 수정 재시도 (최대 3회) · [↓](#21-ensure-대상--검증-재시도) |

---

## 포함 컴포넌트

```
claude-unity-harness/
│
├── skills/                  슬래시 커맨드 구현체 (21개 + 자동 로드 스킬 1개)
│   ├── plan/                설계 플랜 생성
│   ├── review/              코드 리뷰
│   ├── refactor/            리팩토링 플랜 + 실행
│   ├── audit/               전체 감사 (migrate · localize 서브커맨드 포함)
│   ├── perf/                성능 패턴 탐지
│   ├── allow/           권한 자동 승인 토글
│   ├── ship/                전체 개발 파이프라인
│   ├── ensure/              검증 + 자동 재시도 루프
│   ├── fix/                 버그 진단
│   ├── analyze/             코드 분석
│   ├── doc/                 문서 생성 (readme · handover · delivery)
│   ├── git-summary/         커밋 메시지 제안
│   ├── deep-interview/      요구사항 인터뷰
│   ├── context-load/        세션 시작 루틴
│   ├── context-save/        세션 종료 루틴
│   ├── setup/               프로젝트 초기화
│   ├── setup-check/         설치 상태 점검
│   ├── unity-setup-mcp/     Unity MCP 서버 연동 설정
│   ├── unity-console/       Unity 콘솔 에러·경고 분석 (UnityMCP)
│   ├── unity-test/          테스트 실행 + 실패 분석 (UnityMCP)
│   ├── unity-scene-audit/   씬 품질 감사 (UnityMCP)
│   └── unity-patterns/      Unity C# 패턴 레퍼런스 (자동 로드)
│
├── agents/                  스킬이 내부적으로 위임하는 전문 에이전트
│   ├── codebase-explorer    코드베이스 탐색 · 의존 관계 파악
│   ├── architect-planner    설계 플랜 · 리팩토링 플랜 작성
│   ├── unity-reviewer       Unity C# 코드 리뷰
│   ├── doc-writer           문서 작성
│   ├── debugger             버그 진단 (using 누락 자동 검증 포함)
│   ├── critic               대안 제시 · 성능 최적화 관점
│   └── verifier             최종 스펙 검증 · passes:true 판정
│
├── hooks/
│   ├── pre-commit           git commit 시 자동 실행 — .meta 체크·컴파일 검증·코드 리뷰·문서화
│   └── hooks.json           Claude Code 훅 설정
│       ├── SessionStart     세션 시작 시 컨텍스트 자동 주입
│       ├── UserPromptSubmit 컨텍스트 누락 시 자동 보완
│       ├── PreToolUse       .cs 수정 전 Unity 규칙 안내
│       ├── PostToolUse      .cs 저장 후 MCP 연결 시 컴파일 에러 자동 확인 (5초 쿨다운)
│       ├── PreCompact       컨텍스트 압축 전 진행 상황 자동 저장
│       ├── SessionEnd       세션 종료 시 진행 이력 기록
│       └── Stop             세션 메모리 타임스탬프 갱신
│
├── rules/
│   ├── response.md          소통 언어 · 응답 원칙 · 금지 사항
│   ├── git.md               브랜치 전략 · 커밋 메시지 컨벤션
│   └── code-review.md       심각도 기준 (Critical / Warning / Suggestion)
│
├── engines/
│   └── unity.md             Unity 6 LTS 코딩 규칙 · 네이밍 · 생명주기
│
├── languages/
│   └── csharp.md            C# 스타일 가이드 · 포맷 규칙
│
└── domains/
    └── unity.md             Unity 아키텍처 패턴 · UI · 물리 · 씬 관리
```

---

## 케이스별 사용 가이드

### Case 1. 새 Unity 프로젝트를 시작할 때

```
# 1. 플러그인 설치 (최초 1회)
/plugin marketplace add https://github.com/cwp10/claude-unity-harness
/plugin install claude-unity-harness@cwp10-plugins

# 2. Unity 프로젝트 루트에서 초기화
/setup
→ CLAUDE.md에 Unity 버전·스택 자동 기록
→ feature_list.json, claude-progress.txt 생성
→ pre-commit hook 설치

# 3. 만들 기능 목록 정리
/deep-interview 인벤토리 시스템
→ 슬롯 수, 저장 방식, UI 연동 등 3~5개 질문으로 요구사항 확정
→ /plan 자동 연결

# 4. 설계 플랜 확인 후 구현 시작
/plan 인벤토리 시스템
→ Plan Mode 활성화 — 설계 승인 전 파일 수정 차단
→ 클래스 다이어그램 + 대안 2가지 제시
→ 승인 시 코드 구현 시작
```

---

### Case 2. 기존 Unity 프로젝트에 플러그인을 붙일 때

```
# 1. 기존 프로젝트 루트에서 초기화
/setup
→ 기존 코드 건드리지 않음, .claude/ 폴더만 생성

# 2. 현재 코드베이스 파악
/analyze Assets/_Project/Scripts/
→ 시스템 구조·의존 관계·패턴 파악
→ docs/analysis/ 에 저장

# 3. 기술 부채 점검
/audit
→ 전체 .cs 파일 Critical 이슈 스캔
→ .meta 누락, 컴파일 에러 체크

# 4. 이후 일반 개발 흐름으로 진행
```

---

### Case 3. 새 기능을 개발할 때

```
# 단계별 수동 흐름 (각 단계 직접 확인)
/deep-interview 보스 전투 패턴   ← 요구사항 모호할 때
/plan 보스 전투 패턴             ← 설계 플랜 생성 + 승인
/review                          ← 구현 후 코드 리뷰
/context-save                    ← 진행 상황 저장 + git commit
```

```
# 전체 자동화 흐름 (권장 — 설계 승인 1회 후 자동)

# 세션 최초 1회 설정
/allow on  → 세션 재시작

# 이후 매 기능마다 반복 사용
/ship 보스 전투 패턴
→ 설계 플랜 출력 → [승인]
→ 구현 → 리뷰 → 감사 → 검증 자동 진행

# Autopilot이 꺼진 상태로 /ship 실행 시
→ ⚠️ 경고 출력 후 계속 진행 여부 확인
→ 도구 팝업이 발생하지만 나머지 흐름은 동일
```

---

### Case 4. 버그를 수정할 때

```
# Unity 콘솔에서 에러 복사 후
/fix NullReferenceException: Object reference not set
      at PlayerController.Update () (at Assets/.../PlayerController.cs:42)
→ debugger 에이전트가 원인 분석 + 수정 코드 제시
→ using 누락 자동 검증 포함
→ 승인 시 파일 직접 수정

# Unity MCP 연결 시 콘솔 직접 읽기
/unity-console
→ 에러·경고 자동 수집 + 원인 분석
```

---

### Case 5. 코드 품질을 개선할 때

```
# 특정 파일 리뷰
/review Assets/_Project/Scripts/Game/GameManager.cs
→ 성능·Unity 규칙·아키텍처·안전성 4가지 관점 리뷰
→ Critical / Warning / Suggestion 분류

# 대안 설계가 궁금할 때
"더 나은 방법 없어?"
→ critic 에이전트 — 성능 최적화·패턴 대안 제시

# 파일 수가 많아 구조 개선이 필요할 때
/refactor Assets/_Project/Scripts/Combat/
→ Plan Mode 활성화 → 규모 판단 (Small/Medium/Large)
→ Small: 즉시 수정 / Medium·Large: 플랜 승인 후 수정

# 성능 최적화가 필요할 때
/perf Assets/_Project/Scripts/Combat/
→ Update 루프·GC Alloc·Draw Call 위험 패턴 탐지
→ Critical 항목마다 수정 코드 예시 제시
```

---

### Case 6. 작업 도중 세션이 끊겼을 때

```
# 새 세션 시작 시 자동 복구
→ SessionStart hook이 project-memory.json · claude-progress.txt · feature_list.json 자동 주입

# 요약된 형태로 보고 싶을 때
/context-load
→ 마지막 작업, 다음 작업, 알려진 이슈 정리 출력
→ "1번 작업부터 시작할까요?" 제안
```

---

### Case 7. 외주 개발자에게 코드를 넘길 때

```
/doc handover CombatSystem
→ 관련 파일 자동 분석
→ 역할·API·주의사항·의존성 맵 포함 문서 생성
→ docs/guide/HANDOVER_CombatSystem_YYYYMMDD.md 저장
```

---

### Case 8. 클라이언트에 납품할 때

```
# 납품 문서 생성
/doc delivery v1.0.0 2025-06-01
→ feature_list.json에서 구현 기능 자동 추출
→ 실행 방법·제한사항·유지보수 안내 포함
→ docs/guide/DELIVERY_프로젝트명_v1.0.0_20250601.md 저장

# 납품 전 최종 점검
/audit
→ Critical 0건 확인 후 납품
```

---

### Case 9. Unity MCP와 함께 사용할 때

```
# MCP 서버 설치 (최초 1회)
/unity-setup-mcp
→ CoplayDev/unity-mcp 패키지 설치 안내
→ Claude Code 연동까지 단계별 가이드

# 테스트 실행
/unity-test EditMode
→ 전체 EditMode 테스트 실행 + 실패 원인 분석

# 씬 구조 점검
/unity-scene-audit
→ Missing 컴포넌트, 네이밍 위반, 성능 이슈 감사
```

---

## 커맨드 상세

### 1. `/setup` — 프로젝트 초기화

Unity 프로젝트 루트에서 최초 1회 실행합니다.

- `ProjectSettings/ProjectVersion.txt`에서 Unity 버전 자동 감지
- `.claude/CLAUDE.md`에 스택 정보(Unity 버전, 렌더 파이프라인, 패키지) 추가
- `.claude/feature_list.json` 생성 — 기능 진행 현황 추적
- `.claude/claude-progress.txt` 생성 — 세션 이력 기록
- `hooks/pre-commit` hook 설치

---

### 2. `/context-load` — 세션 복구

새 세션 시작 시 이전 작업 상태를 사람이 읽기 좋은 형태로 정리합니다.

- `project-memory.json` + `claude-progress.txt` 내용 요약 출력
- `feature_list.json`에서 `passes: false` 항목 확인 → 다음 할 작업 파악
- `git log --oneline -10` 최근 커밋 이력 출력
- "1번 작업부터 시작할까요?" 다음 작업 제안

> **참고:** SessionStart hook이 세션 시작 시 컨텍스트를 자동 주입합니다. `/context-load`는 그 내용을 사람이 보기 좋게 정리하는 보조 커맨드입니다.

---

### 3. `/context-save` — 세션 저장

작업을 마칠 때 실행합니다. 파일 저장과 git 커밋까지 처리합니다.

1. 현재 대화에서 완료·미완료 작업, 설계 결정, 발견된 문제를 추출합니다.
2. `claude-progress.txt` 업데이트 — 세션 이력 테이블에 오늘 항목 추가.
3. `project-memory.json` 업데이트 — 기술 스택·현재 기능·결정 이력 갱신.
4. `feature_list.json`에서 `passes: true` 항목 확인 + 진행률 반영.
5. `git commit` — 세션 내용에 따라 커밋 타입 자동 선택.

| 세션 내용 | 커밋 타입 |
|----------|----------|
| 새 기능 구현 완료 | `[feat]` |
| 버그 수정 | `[fix]` |
| 리팩토링 | `[refactor]` |
| 문서·분석 파일만 | `[docs]` |
| 설정·하네스 파일 | `[chore]` |

---

### 4. `/deep-interview [기능명]` — 요구사항 명확화

`/plan` 전에 요구사항이 모호할 때 사용합니다.

- 요구사항이 이미 명확하면 → `/plan` 즉시 실행
- 모호하면 → 범위·상호작용·데이터·UI·예외처리 관점에서 3~5개 질문
- 답변 수렴 후 요구사항 요약 → `/plan` 자동 실행

---

### 5. `/plan [기능명]` — 설계 플랜 생성

새 기능 구현 전에 실행합니다. **Plan Mode가 강제 활성화**되어 승인 전에는 파일을 수정할 수 없습니다.

- `codebase-explorer`가 기존 코드베이스에서 유사 패턴·의존 관계 탐색
- `architect-planner`가 규모 판단(Small/Medium/Large), 클래스 다이어그램, 구현 로드맵, 대안 2가지 제시
- 승인 시: `docs/architecture/기능명-설계.md` 저장 + `feature_list.json`에 `passes: false` 항목 추가

---

### 6. `/review [파일]` — 코드 리뷰

파일 경로를 지정하거나 생략 시 변경 파일 자동 감지합니다.

- `codebase-explorer`가 인터페이스·부모 클래스·역참조 탐색 (병렬)
- `unity-reviewer`가 독립 평가자 관점으로 리뷰:
  - 성능: `Update()` 안의 `GetComponent`·`new`·LINQ
  - Unity 규칙: public 필드 노출, Resources.Load, 구형 Input, 이벤트 해제 누락
  - 아키텍처: SRP 위반, 싱글턴 남용, 강한 결합
  - 안전성: null 참조, 씬 전환 참조 유실, CancellationToken 미전달

| 심각도 | 처리 |
|--------|------|
| 🔴 Critical | 즉시 수정 필요 |
| 🟡 Warning | 이번 작업 중 수정 |
| 🟢 Suggestion | 다음 기회에 반영 |

- Critical 0건 시 `verifier` 에이전트로 최종 스펙 검증 권장
- "더 나은 방법 없어?" → `critic` 에이전트가 성능 최적화·대안 패턴 추가 제시

---

### 7. `/audit` — 전체 감사

코드 품질 감사 + 빌드 체크를 병렬로 실행합니다.

- `unity-reviewer`가 변경 파일 전체를 성능·Unity 규칙·아키텍처·안전성 4개 관점으로 분석
- `.meta` 파일 누락 검사
- C# 컴파일 에러 탐지
- 필수 폴더 구조·`.gitignore` 확인

**서브커맨드:**

| 커맨드 | 설명 |
|--------|------|
| `/audit` | 기본 감사 (코드 품질 + 빌드 체크) |
| `/audit migrate` | 구형 Unity API 탐지 — `Input.GetKey`, `Resources.Load`, `StartCoroutine` 등 |
| `/audit localize` | 하드코딩 문자열 탐지 — LocalizationKey 적용 대상 식별 |

---

### 8. `/refactor [대상]` — 리팩토링

**Plan Mode가 강제 활성화**되어 규모 확정 전까지 파일 수정이 차단됩니다.

| 규모 | 기준 | 처리 |
|------|------|------|
| Small | 메서드 추출·네이밍·중복 제거 (1~2개 파일) | Plan Mode 해제 후 즉시 수정 |
| Medium | 클래스 분리·패턴 교체 (3~5개 파일) | 플랜 제시 → 승인 → Plan Mode 해제 → 단계별 수정 |
| Large | 아키텍처 변경·크로스 시스템 (6개+ 파일) | `architect-planner` 위임 |

완료 후 `docs/refactor/이력.md`에 날짜·변경 내용·개선 효과 저장, `verifier`로 컴파일 오류 검증.

---

### 9. `/fix [오류]` — 버그 수정

오류 메시지, 스택 트레이스, 파일 경로 중 하나를 전달합니다.

- 입력 파싱 → `debugger` 에이전트 위임
- `debugger`가 원인 진단 + 수정 코드 제시 (using 누락 자동 검증 포함)
- 승인 시 파일 직접 수정
- 수정 후 컴파일 오류 grep 재검증

---

### 10. `/analyze [대상]` — 코드 분석

코드·기능·시스템을 분석해 사람이 읽기 좋은 형태로 출력합니다.

- `codebase-explorer`가 구조 탐색
- 역할·의존 관계·패턴·개선 포인트 요약
- 저장 여부 확인 → `docs/analysis/` 에 저장 (같은 카테고리 기존 문서 병합)

---

### 11. `/git-summary` — 커밋 메시지 제안

변경사항을 분석해 논리적 커밋 단위와 메시지를 제안합니다.

- `git status` + `git diff --stat` + `git log` 분석
- 변경 파일을 논리적 단위로 분리
- 커밋 타입 자동 선택 + 메시지 제안
- 민감 정보·`.meta` 누락 주의사항 포함
- **커밋 전 컴파일 에러 자동 확인** (MCP 연결 시):
  - 에러 있음 → 커밋 차단 + 에러 목록 출력
  - 에러 없음 → 커밋 진행
  - MCP 미연결 → 수동 확인 요청 후 진행
- "이 단위로 커밋할까요?" 확인 후 실행

---

### 12. `/doc readme|handover|delivery` — 문서 생성

| 서브커맨드 | 생성 위치 | 내용 |
|-----------|----------|------|
| `/doc readme` | `README.md` | 프로젝트 개요·구조·빌드 방법 |
| `/doc handover [시스템명]` | `docs/guide/HANDOVER_*.md` | 외주 개발자용 인수인계 문서 (의존성 맵 Mermaid 포함) |
| `/doc delivery [버전] [날짜]` | `docs/guide/DELIVERY_*.md` | 클라이언트 납품 문서 |

`doc-writer` 에이전트가 소스 코드를 직접 읽고 추측 없이 문서를 작성합니다.

---

### 13. `/setup-check` — 설치 상태 점검

플러그인 설치 상태와 프로젝트 초기화 상태를 진단합니다.

- pre-commit hook 설치 여부
- `CLAUDE.md` 스택 설정 여부
- `feature_list.json` · `claude-progress.txt` 존재 여부
- 문제 발견 시 수정 방법 안내

---

### 14. `/unity-setup-mcp` — Unity MCP 연동

CoplayDev/unity-mcp 서버를 현재 프로젝트에 연동합니다. 연동 후 `/unity-console`, `/unity-test`, `/unity-scene-audit` 사용 가능.

---

### 15. `/unity-console` — 콘솔 분석 _(UnityMCP 필요)_

Unity 콘솔 로그를 읽어 에러·경고를 분류하고 원인 및 수정 방법을 제안합니다.

---

### 16. `/unity-test [EditMode|PlayMode|All]` — 테스트 실행 _(UnityMCP 필요)_

Unity Test Runner를 실행하고 결과를 분석합니다. 실패 항목의 원인과 수정 방법을 제안합니다. 기본값은 EditMode.

---

### 17. `/unity-scene-audit` — 씬 감사 _(UnityMCP 필요)_

현재 Unity 씬의 구조를 분석하여 품질 이슈를 보고합니다.

- Missing 컴포넌트 탐지
- 네이밍 규칙 위반 (PFB\_, MAT\_ 등)
- 성능 이슈 (과도한 Draw Call, 비활성 오브젝트 등)
- 구조적 문제 (중첩 깊이, 빈 GameObject 등)

---

### 18. `/perf [파일]` — 성능 분석

파일·폴더를 지정하거나 생략 시 최근 변경 파일을 자동 탐지합니다.

Update 루프·GC Alloc·Draw Call 위험 패턴을 탐지합니다.

| 탐지 항목 | 심각도 |
|---------|--------|
| `Update` 내 `GetComponent` 호출 | 🔴 Critical |
| `Update` 내 `Instantiate/Destroy` (ObjectPool 미사용) | 🔴 Critical |
| `Update` 내 `new` · LINQ · `string+` | 🔴 Critical |
| `FindObjectOfType` — `Update` 외부 사용 | 🟡 Warning |
| 이벤트 구독 후 해제 누락 위험 | 🟡 Warning |

Critical 항목마다 수정 코드 예시를 함께 제시합니다.

> 전체 프로젝트 성능 점검은 `/audit`을, 특정 파일·시스템 심층 분석은 `/perf`를 사용하세요.

---

### 19. `/allow [on|off]` — 오토파일럿

`.claude/settings.json`의 권한 설정을 토글합니다. 인수 없이 실행하면 현재 상태를 확인합니다.

| 커맨드 | 동작 |
|--------|------|
| `/allow` | 현재 ON/OFF 상태 + allow·deny 목록 출력 |
| `/allow on` | `Bash(*)`·`Edit(*)`·`Write(*)` 자동 승인 추가 |
| `/allow off` | 추가된 자동 승인 항목 제거, 기본 승인 방식 복원 |

**ON 시 권한 구성:**
- 자동 승인: `Bash(*)`·`Edit(*)`·`Write(*)`
- 항상 차단: `Bash(git push*)` — 푸시는 오토파일럿 상태와 무관하게 항상 확인

> 기존 allow·deny 항목은 건드리지 않습니다. 오토파일럿이 추가한 항목만 넣고 뺍니다.
> 변경 후 세션을 재시작해야 적용됩니다.

**`/ship` · `/ensure`와 연계:**

`/ship`과 `/ensure`는 시작 시 자동으로 Autopilot 상태를 확인합니다. OFF 상태이면 경고를 출력하고, 켜는 방법을 안내합니다. 완전 자동화를 원한다면 세션 시작 시 `/allow on` → 재시작 후 사용하세요.

```
# 권장 설정 흐름
/allow on      ← 세션 시작 시 1회
→ 세션 재시작
→ /ship 기능명     ← 이후 매번 사용 가능
```

---

### 20. `/ship [기능명]` — 전체 파이프라인

**설계 플랜 승인 1회** 후 구현→리뷰→감사→검증까지 자동으로 진행합니다.

시작 시 Autopilot 상태를 자동 확인합니다. OFF이면 경고 후 `/allow on` → 재시작을 안내합니다.

| 단계 | 내용 | 승인 |
|------|------|------|
| 0. Autopilot 확인 | OFF이면 경고 출력 | — |
| 1. 요구사항 확인 | 모호하면 `/deep-interview` 자동 연결 | — |
| 2. 설계 플랜 | `/plan --ship` — Plan Mode 강제, 클래스 다이어그램 + 대안 2가지 | **필요** |
| 3. 코드 구현 | 승인된 플랜 기반 구현 (`--ship` 플래그로 구현 시작 자동 통과) | 자동 |
| 4. 코드 리뷰 | `/review` — 변경 파일 전체 대상, Critical 있으면 `/fix` 자동 (최대 2회) | 자동 |
| 5. 전체 감사 | `/audit` — Critical 있으면 `/fix` 자동 (1회) | 자동 |
| 6. 최종 검증 | `/ensure` — verifier + 자동 재시도 (최대 3회) | 자동 |

---

### 21. `/ensure [대상]` — 검증 재시도

verifier 검증 → 실패 시 자동 수정 → 재검증을 최대 3회 반복합니다. `/ship` 마지막 단계에서 자동 호출되거나 단독으로 사용합니다.

```
verifier 실행 → PASS → 완료 ✅
             → FAIL → debugger 자동 수정 → 재검증
                    → FAIL → 재수정 → 재검증
                           → FAIL → 이슈 리포트 출력, 사용자에게 넘김 ⛔
```

3회 통과 못하면 `/plan`으로 설계 재검토를 권장합니다.

---

## 요구 사항

- Claude Code CLI
- Unity 6 LTS
- Git (Windows: Git Bash / macOS: 기본 터미널)
