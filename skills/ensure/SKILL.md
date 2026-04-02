---
name: ensure
description: >
  스펙 검증 + 실패 시 자동 수정 재시도 루프 (최대 3회).
  verifier가 PASS할 때까지 자동 수정 → 재검증을 반복합니다.
  /ship 마지막 단계에서 자동 호출되거나 단독으로 사용합니다.
  Usage: /ensure [파일 또는 기능명] (기본: 최근 변경 파일)
  Example: /ensure
  Example: /ensure Assets/_Project/Scripts/Combat/BossController.cs
tools: Read, Glob, Grep, Bash, Write, Edit
model: sonnet
triggers:
  - "검증하고 고쳐줘"
  - "완료될 때까지"
  - "자동으로 고쳐서 완료"
  - "ensure"
  - "통과될 때까지"
keywords: [ensure, verify, retry, fix, guarantee, completion]
---

대상: $ARGUMENTS

## Ensure — 검증 + 자동 재시도

최대 3회까지 verifier 검증 → 실패 시 자동 수정을 반복한다.
3회 모두 실패하면 이슈 리포트를 출력하고 사용자에게 넘긴다.

---

## 실행 순서

### Autopilot 상태 확인

`.claude/settings.json`을 읽어 `permissions.allow`에 `"Bash(*)"` 포함 여부 확인.

> `/ship`에서 호출된 경우 이 체크를 생략한다 (이미 0단계에서 확인됨).

**OFF 상태이면:**
```
⚠️  Autopilot이 꺼져 있습니다.

검증·수정 중 Write·Edit·Bash 도구 사용 시 매번 승인 팝업이 발생합니다.
완전 자동화를 원하면:

  1. /autopilot on
  2. 세션 재시작
  3. /ensure 재실행

지금 그대로 진행하려면 계속하세요.
```
→ 사용자가 계속 진행하면 그대로 검증 루프 시작.

**ON 상태이면:** 아무 메시지 없이 검증 루프 바로 시작.

---

### 시작 전 준비

대상 파악:
- `$ARGUMENTS` 있음 → 지정 파일·기능 대상
- 없음 → `git diff --name-only HEAD`로 최근 변경 .cs 파일 대상
- 변경 파일도 없으면 → "검증할 대상을 지정해 주세요" 질문

스펙 파악 (우선순위):
1. `docs/architecture/` 에서 관련 설계 문서 Read
2. `.claude/feature_list.json` 에서 해당 기능 항목 확인
3. 위 모두 없으면 → 대화 컨텍스트에서 스펙 추론 후 사용자 확인

---

### 검증 루프 (최대 3라운드)

**라운드 시작 시 출력:**
```
🔄 Ensure 검증 [N/3회] — [대상 파일명]
```

**[검증] verifier 에이전트 위임:**
- 코드 품질 (Critical 없음)
- 스펙 달성 여부
- stub/TODO 잔존 여부

**[판정]**

PASS → 루프 종료, 성공 출력

FAIL →
```
❌ 검증 실패 [N/3회]
이슈:
- [이슈 1]
- [이슈 2]
자동 수정을 시도합니다...
```

→ debugger 에이전트에게 위임:
  - 실패 이슈 목록 전달
  - 파일 직접 수정 요청
  - using 누락 자동 검증 포함

→ 수정 완료 후 다음 라운드 검증 반복

---

### 3회 모두 실패 시

```
⛔ Ensure 완료 불가 — 3회 시도 후에도 미해결 이슈 있음

## 미해결 이슈 목록
[라운드별 이슈 요약]

## 권장 조치
- 설계 재검토 필요: /plan [기능명]
- 수동 수정 후 재시도: /ensure [파일]
- 긴급 커밋이 필요하면: git commit --no-verify
```

feature_list.json `passes: false` 유지.

---

### PASS 시 출력

```
✅ Ensure 완료 — [N]회 시도만에 통과

검증 결과:
  코드 품질: ✅ Critical 0건
  스펙 달성: ✅ 모든 요구사항 충족
  stub/TODO: ✅ 없음

feature_list.json → passes: true 갱신 완료.
/context-save 로 진행 상황을 저장하는 것을 권장합니다.
```

verifier 에이전트가 `.claude/feature_list.json`의 해당 기능 `passes: true`로 변경.
