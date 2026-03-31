---
name: web-review-rules
description: >
  Provides Next.js / TypeScript code review criteria and output format for detecting
  type safety, architecture, performance, and security violations.
  Loads automatically when reviewing, checking, or inspecting web/TypeScript code.
  Do NOT use for Unity, C#, or non-web code review.
allowed-tools: Read, Grep, Glob
user-invocable: false
---

# 웹 코드 리뷰 스킬

## 분석 체크리스트
상세 항목: [references/checklist.md](references/checklist.md)

핵심 🔴 Critical (즉시 기억):
- `'use client'` 파일에서 DB 직접 접근·서버 전용 패키지 import → 아키텍처
- Server Component에서 `useState`·`useEffect` 사용 → Next.js 규칙
- 인증 없는 API Route → 보안
- 환경변수 클라이언트 노출 (`NEXT_PUBLIC_` 오남용) → 보안

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
