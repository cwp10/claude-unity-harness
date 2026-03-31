---
name: architect-output
description: >
  Provides output format templates for Small, Medium, and Large architecture plans,
  and docs/architecture/ document structure for the architect agent.
  Loads automatically when the architect agent produces design documents.
  Do NOT use directly — loaded by the architect agent as a reference.
allowed-tools: Read, Write
user-invocable: false
---

# 설계 산출물 출력 형식

상세 템플릿: [references/templates.md](references/templates.md)

## 규모별 핵심 요약

| 규모 | 산출물 |
|------|--------|
| Small | 변경 위치 + 구현 포인트 + 주의사항 |
| Medium | Small + 다이어그램 + 파일 구성 + 로드맵 + 대안 2가지 |
| Large | Medium + 영향 범위 + 마이그레이션 플랜 + 롤백 계획 |

## 공통 원칙
- 대안은 항상 2가지 이상, 트레이드오프 명시
- 승인 전 코드 작성 절대 금지
- 승인 후 `docs/architecture/기능명-설계.md` 자동 저장
