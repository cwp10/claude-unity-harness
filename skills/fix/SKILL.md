---
name: fix
description: >
  Diagnoses errors and bugs in Unity C# code by analyzing stack traces, error messages, and related code.
  Usage: /fix [오류 메시지 또는 파일경로]
  Example: /fix NullReferenceException at PlayerController.cs:42
  Example: /fix Assets/_Project/Scripts/Game/PlayerController.cs
  Provides root cause analysis and corrected code.
  Do NOT use for new feature development or code review — use /plan or /review.
tools: Read, Glob, Grep, Bash, Write
model: sonnet
triggers:
  - "에러"
  - "버그"
  - "오류"
  - "NullReferenceException"
  - "왜 안 돼"
  - "크래시"
keywords: [fix, debug, error, bug, stacktrace, exception, unity]
---

대상: $ARGUMENTS

## 실행 순서

### 1단계: 입력 파악

입력 분석:
- 스택 트레이스 포함 → 파일명·줄번호 추출
- 파일경로 포함 → 해당 파일 직접 분석
- 오류 메시지만 → 키워드로 관련 .cs 파일 Grep 탐색
- 아무것도 없음 → "어떤 오류인지 설명해 주세요" 질문

### 2단계: debugger 에이전트 위임

debugger 에이전트에게 다음을 전달해서 진단·수정까지 위임한다:
- 오류 메시지 / 스택 트레이스
- 관련 파일 경로
- 프로젝트 타입: Unity C#

### 3단계: 하네스 게이트 — 수정 후 검증

debugger가 파일 수정을 완료하면 반드시 아래 검증을 수행한다:

```bash
grep -r "CS[0-9]\{4\}" . --include="*.cs" 2>/dev/null | head -5
```

검증 결과 판정:

| 결과 | 처리 |
|------|------|
| 컴파일 오류 없음 | `/review` 로 품질 검증 후 `verifier` 에이전트로 최종 passes:true 처리 권장 |
| 컴파일 오류 있음 | `passes: false` 유지 + 오류 목록 출력 + debugger 재위임 |
