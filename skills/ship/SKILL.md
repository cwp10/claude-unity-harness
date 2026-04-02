---
name: ship
description: >
  Unity 기능 개발 전체 파이프라인을 순차 실행합니다.
  deep-interview → plan → 구현 → review → audit → ensure(검증+재시도) 까지 자동 진행.
  설계 플랜 승인 후 구현부터 검증까지는 자동으로 진행됩니다.
  Usage: /ship [기능명]
  Example: /ship 인벤토리 시스템
  Example: /ship 보스 전투 패턴
tools: Read, Glob, Grep, Bash, Write, Edit
model: sonnet
triggers:
  - "전체 파이프라인"
  - "처음부터 끝까지"
  - "자동으로 개발"
  - "워크플로우"
  - "ship"
keywords: [ship, pipeline, workflow, end-to-end, development]
---

기능명: $ARGUMENTS

## Ship 개요

```
1단계: 요구사항 확인       (자동)
2단계: 설계 플랜           (승인 필요 — 이후 자동)
3단계: 코드 구현           (자동)
4단계: 코드 리뷰           (자동, Critical 시 자동 fix)
5단계: 전체 감사           (자동, Critical 시 자동 fix)
6단계: 검증 + 재시도       (자동, /ensure)
```

**승인 포인트는 2단계(설계 플랜) 1회뿐입니다.**
승인 후 구현 → 리뷰 → 감사 → 검증까지 자동으로 진행합니다.

---

## 1단계: 요구사항 확인

`$ARGUMENTS`가 없으면 → "어떤 기능을 만들까요?" 질문 후 입력받는다.

요구사항이 모호한지 판단:
- **명확함** → 2단계 바로 진행
- **모호함** → `/deep-interview $ARGUMENTS` 실행 후 요구사항 확정

---

## 2단계: 설계 플랜 ← 유일한 승인 포인트

`/plan $ARGUMENTS` 실행.

- codebase-explorer가 기존 코드베이스 탐색
- architect-planner가 클래스 다이어그램 + 대안 2가지 제시
- Plan Mode 강제 활성화 — 승인 전 파일 수정 차단

플랜 출력 후:
```
📋 설계 플랜 완료.
승인하면 구현 → 리뷰 → 감사 → 검증까지 자동으로 진행됩니다.
[승인 / 수정 요청 / 중단]
```

승인 시 → 3단계부터 자동 진행
수정 요청 시 → 플랜 수정 후 재승인
중단 시 → 종료, 플랜은 docs/architecture/ 에 저장됨

---

## 3단계: 코드 구현 (자동)

승인된 설계 플랜 기반으로 코드를 구현한다.

- docs/architecture/ 에 저장된 설계 문서 참조
- Unity 코딩 규칙 (engines/unity.md) 준수
- feature_list.json 에 `passes: false` 항목 추가

완료 후 → 4단계 자동 진행

---

## 4단계: 코드 리뷰 (자동)

`/review` 실행 (변경 파일 자동 감지).

| 결과 | 처리 |
|------|------|
| Critical 0건 | 5단계 자동 진행 |
| Critical 1건 이상 | `/fix` 자동 실행 → 재리뷰 (최대 2회) |
| 2회 후에도 Critical 있음 | 이슈 목록 출력 + 사용자 판단 요청 후 중단 |

---

## 5단계: 전체 감사 (자동)

`/audit` 실행.

| 결과 | 처리 |
|------|------|
| Critical 0건 | 6단계 자동 진행 |
| Critical 1건 이상 | `/fix` 자동 실행 → 재감사 (1회) |
| 재감사 후에도 Critical 있음 | 이슈 목록 출력 + 사용자 판단 요청 후 중단 |

---

## 6단계: 최종 검증 (자동, /ensure)

`/ensure` 실행.

- verifier 에이전트가 스펙 충족 여부 독립 검증
- FAIL 시 자동 수정 재시도 (최대 3회)
- 상세 동작은 /ensure 스킬 참조

**PASS:**
```
🎉 Ship 완료!

기능명: [기능명]
  ✅ 요구사항 확인
  ✅ 설계 플랜 (승인됨)
  ✅ 코드 구현
  ✅ 코드 리뷰
  ✅ 전체 감사
  ✅ 최종 검증 PASS

/context-save 로 진행 상황을 저장하는 것을 권장합니다.
```

**최종 FAIL:**
```
⚠️ Ship 미완료.

[미해결 이슈 목록]

수동 수정 후 /ensure 로 재검증하세요.
```
