# claude-unity 플러그인

Unity 프로젝트를 위한 Claude Code 전용 플러그인.

세션 관리 하네스 + 코드 리뷰 + 설계 플랜 + 문서화 + pre-commit 자동 리뷰를 제공합니다.

---

## 포함 컴포넌트

### Skills (슬래시 커맨드)

| 커맨드 | 설명 |
|--------|------|
| `/setup` | Unity 프로젝트 초기화 (최초 1회) |
| `/context-load` | 세션 시작 — 진행 상황 복구 |
| `/context-save` | 세션 종료 — 진행 상황 저장 + 커밋 |
| `/plan [기능명]` | 설계 플랜 생성 (codebase-explorer → architect-planner) |
| `/review [파일]` | 코드 리뷰 (unity-reviewer 에이전트) |
| `/audit` | 전체 감사 + 빌드 체크 |
| `/refactor [대상]` | 리팩토링 플랜 + 실행 |
| `/debug [오류]` | 버그 진단 + 수정 |
| `/analyze [대상]` | 코드·시스템 분석 |
| `/git-summary` | 커밋 메시지 제안 |
| `/doc readme\|handover\|delivery` | 문서 생성 |

### Reference Skills (자동 로드)

| 스킬 | 설명 |
|------|------|
| `unity-patterns` | Unity C# 패턴 코드 레퍼런스 |
| `unity-review-rules` | 코드 리뷰 체크리스트 |
| `git-workflow` | Git 커밋 컨벤션 |
| `architect-output` | 설계 산출물 템플릿 |
| `doc-templates` | 문서 템플릿 |

### Agents (자동 호출)

| 에이전트 | 역할 |
|----------|------|
| `unity-reviewer` | Unity C# 독립 코드 평가자 |
| `architect-planner` | 설계 플랜 + 리팩토링 플랜 |
| `codebase-explorer` | 코드베이스 구조 탐색 (읽기 전용) |
| `debugger` | 버그 진단 + 수정 |
| `doc-writer` | 문서 작성 |

### Hooks

| 훅 | 동작 |
|----|------|
| `hooks/pre-commit` | `.meta` 확인 → Unity 컴파일 → 코드 리뷰 → 문서 자동화 |

---

## 설치 방법

### Claude Code CLI

```bash
# 플러그인 설치
claude plugin install claude-unity.plugin

# 프로젝트 초기화 (Unity 프로젝트 루트에서)
/setup
```

### pre-commit hook 수동 설치

`/setup` 실행 시 자동 설치됩니다.
수동 설치 시:
```bash
cp .claude/hooks/pre-commit .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

---

## 워크플로우

```
새 세션 시작
  → /context-load          (진행 상황 복구)

기능 개발
  → /plan [기능명]          (설계 플랜)
  → 코드 작성
  → /review [파일]          (코드 리뷰)
  → /audit                  (전체 감사)

세션 종료
  → /context-save           (저장 + 커밋)

커밋 시 자동 실행
  → pre-commit hook
     [0] .meta 확인
     [1] Unity 컴파일 검증
     [2] 코드 리뷰 (Critical 있으면 차단)
     [3] 문서 자동 생성
```

---

## 요구 사항

- Claude Code CLI
- Unity 6 LTS (6000.0.x)
- Git for Windows (Git Bash)
- claude CLI가 PATH에 등록되어 있어야 pre-commit 리뷰 동작

---

## 적용 프로젝트 유형

- 게임 (모바일/PC)
- 산업 시뮬레이션
- 교육 콘텐츠 (VR/AR 포함)
