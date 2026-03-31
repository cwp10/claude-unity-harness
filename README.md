# claude-unity

Unity 프로젝트를 위한 Claude Code 플러그인.

세션 관리 하네스, 코드 리뷰, 설계 플랜, 문서화, pre-commit 자동 리뷰를 제공합니다.

---

## 설치

```bash
claude plugin install claude-unity.plugin
```

설치 후 Unity 프로젝트 루트에서 최초 1회 실행:

```bash
/setup
```

---

## 슬래시 커맨드

| 커맨드 | 설명 |
|--------|------|
| `/setup` | 프로젝트 초기화 — CLAUDE.md 스택 설정, 하네스 파일 생성, hook 설치 |
| `/context-load` | 세션 시작 — 이전 진행 상황 복구 |
| `/context-save` | 세션 종료 — 진행 상황 저장 + git 커밋 |
| `/plan [기능명]` | 설계 플랜 생성 |
| `/review [파일]` | 코드 리뷰 |
| `/audit` | 전체 감사 + 빌드 체크 |
| `/refactor [대상]` | 리팩토링 플랜 + 실행 |
| `/debug [오류]` | 버그 진단 + 수정 |
| `/analyze [대상]` | 코드·시스템 분석 |
| `/git-summary` | 커밋 메시지 제안 |
| `/doc readme\|handover\|delivery` | 문서 생성 |

---

## 워크플로우

```
새 세션
  /context-load        ← 진행 상황 복구

기능 개발
  /plan [기능명]        ← 설계 플랜
  코드 작성
  /review [파일]        ← 코드 리뷰
  /audit               ← 전체 감사

세션 종료
  /context-save        ← 저장 + 커밋

git commit 시 자동 실행
  [0] .meta 파일 확인
  [1] Unity 컴파일 검증
  [2] 코드 리뷰 (Critical 발견 시 차단)
  [3] 문서 자동 생성
```

---

## 포함 컴포넌트

```
claude-unity-plugin/
├── skills/           슬래시 커맨드 11개 + 레퍼런스 스킬 5개
├── agents/           unity-reviewer, architect-planner,
│                     codebase-explorer, debugger, doc-writer
├── hooks/            pre-commit (Unity 자동 리뷰)
├── rules/            응답·git·코드리뷰 규칙
├── engines/          Unity 6 LTS 코딩 규칙
├── languages/        C# 스타일 가이드
└── domains/          Unity 아키텍처 패턴
```

---

## 요구 사항

- Claude Code CLI
- Unity 6 LTS
- Git for Windows (Git Bash)

---

## 플러그인 빌드

```powershell
cd claude-unity-plugin
Compress-Archive -Path * -DestinationPath ..\claude-unity-temp.zip
Rename-Item ..\claude-unity-temp.zip ..\claude-unity.plugin
```
