---
name: plan
description: >
  Orchestrates codebase-explorer then architect-planner agents in sequence.
  codebase-explorer maps the existing codebase first, then passes findings
  to architect-planner to produce a grounded architecture plan.
  Usage: /plan [feature description]
  Example: /plan 인벤토리 시스템
  Example: /plan 보스 전투 패턴
tools: Read, Glob, Grep, Write
model: opus
triggers:
  - "설계해줘"
  - "아키텍처"
  - "어떻게 만들까"
  - "플랜 짜줘"
  - "구조 잡아줘"
keywords: [plan, architecture, design, feature, system]
---

요청: $ARGUMENTS

## 실행 순서

### 1단계: 병렬 현황 파악

아래 2개 작업을 동시에(concurrently) 실행한다:

**Task A — codebase-explorer 에이전트:**
- 요청 기능과 관련된 기존 파일·폴더 구조 탐색
- 유사한 기존 시스템 패턴 파악
- 관련 클래스·컴포넌트 의존 관계 요약

**Task B — 직접 실행 (메인 Claude):**
- `docs/architecture/` 에서 관련 설계 문서 확인
- `docs/analysis/` 에서 관련 분석 문서 확인

두 작업이 완료되면 결과를 합쳐 2단계로 전달한다.

### 2단계: architect-planner 에이전트로 설계 플랜 생성

1단계 탐색 결과를 컨텍스트로 포함하여 architect-planner 에이전트에게 위임:

```
요청: $ARGUMENTS

[codebase-explorer 탐색 결과]
{1단계 결과 전체}

위 탐색 결과를 바탕으로 설계 플랜을 작성해줘.
- 기존 패턴과 일관된 설계 제안
- 규모 판단 (Small / Medium / Large)
- 클래스·컴포넌트 다이어그램
- 구현 로드맵
- 대안 2가지 + 트레이드오프
- 승인 전 코드 작성 금지
```

### 3단계: 승인 및 후속 실행

architect-planner 의 설계 플랜을 그대로 출력한다.
"이 설계로 진행할까요?" 확인 요청 후 대기.

승인 시:
1. architect-planner 가 docs/architecture/기능명-설계.md 자동 저장
2. .claude/feature_list.json 에 해당 기능 항목 추가 (`passes: false` 로 시작)
3. "코드 구현을 시작할까요?" 확인
4. 확인 시 메인 Claude가 구현 플랜대로 단계별 코드 생성
