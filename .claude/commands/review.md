---
description: >
  Orchestrates codebase-explorer then unity-reviewer in sequence.
  codebase-explorer maps related files first, then review is performed
  with full context of dependencies and related classes.
  Usage: /review [filepath]
  Example: /review Assets/_Project/Scripts/Game/PlayerController.cs
  If no file specified, auto-detects recently changed .cs files.
allowed-tools: Read, Grep, Glob, Bash, Write
---

대상 파일: $ARGUMENTS

## 실행 순서

### 1단계: 리뷰 대상 결정

파일이 명시된 경우 → 해당 파일로 확정
파일이 없는 경우:
1. `git diff --name-only HEAD` 로 변경 파일 확인
2. `.cs` 파일만 필터링
3. 여러 개면 목록 보여주고 선택 요청

### 2단계: 병렬 컨텍스트 수집

아래 2개 작업을 동시에(concurrently) 실행한다:

**Task A — codebase-explorer 에이전트:**
- 리뷰 대상 파일이 참조하는 인터페이스·부모 클래스·ScriptableObject 탐색
- 이 파일을 참조하는 다른 파일 탐색 (역참조)
- 관련 이벤트·메시지 구독 관계 파악

**Task B — 직접 실행 (메인 Claude):**
- `git log --oneline -5 [대상 파일]` 로 최근 변경 이력 확인
- `docs/analysis/` 에서 관련 분석 문서 확인

두 작업이 완료되면 결과를 합쳐 3단계로 전달한다.

### 3단계: unity-reviewer 에이전트에 위임

이 단계는 코드를 생성한 에이전트와 분리된 독립 평가다.
회의적 기본 자세로 임한다 — 코드가 틀렸다고 가정하고 검증한다.

2단계 탐색 결과를 컨텍스트로 포함하여 unity-reviewer 에이전트에 위임:
- 성능: Update 안의 GetComponent/Find, 매 프레임 new, LINQ, 오브젝트 풀 미적용
- Unity 규칙: public 필드 노출, Magic number, Resources.Load(), 구형 Input, 이벤트 해제 누락
- 아키텍처: SRP 위반, 싱글턴 남용, 강한 결합 (탐색된 의존 관계 기반으로 판단)
- 안전성: null 참조, 씬 전환 참조 유실, Destroy 후 접근, CancellationToken 미전달

### 4단계: 출력 형식

```
## 코드 리뷰 결과: [파일명]

### 요약
전체 평가 한 줄. Critical 유무 명시.

### 발견 이슈

🔴 [Critical] 줄번호
문제: 설명
현재 코드: (문제 코드)
수정 코드: (수정된 코드)

🟡 [Warning] 줄번호
문제: 설명
수정 방향: 설명 또는 코드 예시

🟢 [Suggestion] 줄번호
제안: 설명

### 통계
| 심각도 | 건수 |
|--------|------|
| 🔴 Critical | N건 |
| 🟡 Warning | N건 |
| 🟢 Suggestion | N건 |

### 우선 수정 순서
1. (가장 중요한 것부터)
```

### 주의
- 이슈가 없는 항목은 출력 생략
- Critical 0건이면 요약에 명시
- 수정 코드는 실제 동작하는 코드로 (의사코드 금지)
- 이슈 20건 초과 시 Critical·Warning만 출력, Suggestion 생략
- 탐색된 의존 관계를 활용해 파일 단독으로는 파악 불가능한 문제도 진단

### 5단계: 하네스 게이트

리뷰 결과를 받아 판정:

| 결과 | 처리 |
|------|------|
| Critical 0건 | feature_list.json 해당 기능 `passes: true` 로 변경 후 완료 보고 |
| Critical 1건 이상 | `passes: false` 유지 + 커밋 차단 안내 + 우선 수정 항목 목록 출력 |
