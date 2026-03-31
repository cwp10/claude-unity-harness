---
name: unity-test
description: >
  Unity Test Runner를 실행하고 결과를 분석합니다. 실패 항목의 원인과 수정 방법을 제안합니다.
  UnityMCP run_tests 툴을 사용합니다.
  Usage: /unity-test [EditMode|PlayMode|All] (기본: EditMode)
allowed-tools: mcp__UnityMCP__run_tests, mcp__UnityMCP__read_console, mcp__UnityMCP__find_in_file, Read, Glob, Grep
triggers:
  - "테스트 실행"
  - "테스트 돌려줘"
  - "유닛 테스트"
keywords: [test, unittest, editmode, playmode, runner]
---

# /unity-test

Unity 테스트를 실행하고 결과를 분석한다.

## 인자 파싱

사용자 입력에서 플랫폼 파싱:
- `EditMode` (기본값) — 에디터 모드 테스트
- `PlayMode` — 플레이 모드 테스트
- `All` — 전체 실행

## MCP 연결 확인

UnityMCP `run_tests` 툴을 호출한다.

**미연결 시:**
```
[unity-test] UnityMCP가 연결되지 않았습니다.

해결 방법:
1. Unity Editor 실행
2. Window > MCP For Unity 열기
3. Start Server 클릭
4. /unity-test 재실행
```
→ 여기서 종료.

## 실행 순서

### 1단계: 테스트 실행

`run_tests` 툴 호출 — 선택한 플랫폼으로 실행.

실행 중 메시지:
```
⏳ Unity [EditMode|PlayMode] 테스트 실행 중...
```

### 2단계: 결과 요약 출력

```
## Unity 테스트 결과 — EditMode

✅ 통과: N건
❌ 실패: N건
⚪ 생략: N건
⏱️ 소요: Xs
```

### 3단계: 실패 항목 상세 분석

실패 항목이 있으면 각 항목:

```
### ❌ [테스트명]
클래스: TestClassName.MethodName

실패 메시지:
  Expected 10 but was 5
  at PlayerController.TakeDamage (line 42)

원인: (실패 원인 분석 — 테스트 코드와 구현 코드 불일치, 로직 버그 등)

수정 방법:
  (구체적인 수정 코드 또는 방향 제시)
```

실패 항목을 수정하기 전에 관련 파일을 Read로 읽어 정확한 원인을 파악한다.

### 4단계: 수정 제안

실패 0건:
```
✅ 모든 테스트 통과. 코드 품질이 검증됐습니다.
verifier로 passes: true 처리를 진행하세요.
```

실패 N건:
```
## 수정 우선순위
1. [가장 먼저 수정할 테스트] — 이유
2. ...

수정 후 /unity-test 를 다시 실행하세요.
```
