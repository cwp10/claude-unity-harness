---
name: refactor
description: >
  Analyzes existing code and produces a refactoring plan with step-by-step execution.
  Usage: /refactor [파일 또는 시스템명]
  Example: /refactor CombatSystem
  Example: /refactor Assets/_Project/Scripts/Game/Combat/
  Detects scale (Small/Medium/Large) and either fixes directly or presents a plan first.
  Saves refactoring history to docs/refactor/ on completion.
  Do NOT use for new feature development — use /plan instead.
allowed-tools: Read, Glob, Grep, Write, Bash
cost: opus
triggers:
  - "리팩토링"
  - "코드 정리"
  - "개선해줘"
  - "구조 개선"
  - "중복 제거"
keywords: [refactor, cleanup, architecture, solid, pattern]
---

대상: $ARGUMENTS

## 실행 순서

### 1단계: codebase-explorer + architect-planner 파이프라인

codebase-explorer 에이전트에게 위임:
- 대상 파일/시스템의 현재 구조 파악
- 코드 품질 문제점 목록 수집 (중복·복잡도·결합도)
- 이 코드를 참조하는 다른 파일 파악 (역참조 — 영향 범위)
- 기존 리팩토링 이력 확인 (`docs/refactor/` 폴더)

탐색 결과를 보관한다.

### 2단계: 리팩토링 규모 판단

| 규모 | 기준 | 처리 |
|------|------|------|
| Small | 메서드 추출·네이밍·중복 제거 (1~2개 파일) | 즉시 수정 |
| Medium | 클래스 분리·패턴 교체 (3~5개 파일) | 플랜 제시 → 승인 → 단계별 수정 |
| Large | 아키텍처 변경·크로스 시스템 (6개+ 파일) | architect-planner 에이전트 위임 |

### 3단계: 규모별 처리

**Small — 즉시 수정:**
수정 내용 설명 후 파일 직접 수정.
수정 완료 후 docs/refactor/이력.md 업데이트.

**Medium/Large — architect-planner 에이전트 위임:**
탐색 결과를 architect-planner 에이전트에게 전달.
architect-planner 가 리팩토링 플랜 형식으로 출력 후 승인 대기.

승인 후:
- 메인 Claude가 플랜대로 단계별 수정 실행
- 각 단계 완료 후 사용자 확인

### 4단계: 완료 후 이력 저장

`docs/refactor/` 폴더에 이력 저장:
```markdown
# [대상] 리팩토링 이력

날짜: YYYY-MM-DD
규모: Small / Medium / Large

## 변경 내용
(주요 변경사항)

## 변경 파일
(파일 목록)

## 개선 효과
(before/after 핵심 차이)
```

### 5단계: 하네스 게이트 — 리팩토링 검증

이력 저장 후 반드시 아래 검증을 수행한다:

```bash
grep -rn "CS[0-9]\{4\}" . --include="*.cs" 2>/dev/null | head -20
```

검증 결과 판정:

| 결과 | 처리 |
|------|------|
| 오류 없음 | feature_list.json 해당 기능 `passes: true` 로 변경 + 완료 보고 |
| 오류 있음 | `passes: false` 유지 + 오류 목록 출력 + 수정 재진행 |
