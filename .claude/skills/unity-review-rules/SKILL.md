---
name: unity-review-rules
description: >
  Provides Unity C# code review criteria and output format for detecting
  performance, safety, architecture, and Unity rule violations.
  Loads automatically when reviewing, checking, or inspecting Unity C# code.
  Do NOT use for web, TypeScript, or non-Unity code review.
allowed-tools: Read, Grep, Glob
user-invocable: false
---

# 코드 리뷰 스킬

## 분석 체크리스트
상세 항목: [references/checklist.md](references/checklist.md)

핵심 🔴 Critical (즉시 기억):
- Update 안 GetComponent/Find/new → 성능
- 이벤트 구독 해제 누락 → 메모리 릭
- null 참조·씬 전환 유실·Destroy 후 접근 → 안전성

---

## 심각도 기준

rules/code-review.md 참조.

---

## 출력 형식
```
## 코드 리뷰 결과: [파일명]

### 요약
전체 평가 한 줄. Critical 유무 명시.

### 발견 이슈

🔴 [Critical] 줄번호
문제: 설명
현재 코드:
(문제 코드)
수정 코드:
(수정된 코드)

🟡 [Warning] 줄번호
문제: 설명
수정 방향: 설명

🟢 [Suggestion] 줄번호
제안: 설명

### 통계
🔴 Critical: N건 / 🟡 Warning: N건 / 🟢 Suggestion: N건

### 우선 수정 순서
1. (가장 중요한 Critical부터)
```

## 주의
- 이슈 없는 항목은 출력 생략
- 수정 코드는 실제 동작하는 코드로 (의사코드 금지)
- 이슈 20건 초과 시 Critical·Warning만 출력
