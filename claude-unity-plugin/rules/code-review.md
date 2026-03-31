# 코드 리뷰 규칙

## 심각도 기준

| 심각도 | 기준 | 처리 |
|--------|------|------|
| 🔴 Critical | 빌드 실패·크래시·메모리 릭·데이터 손실·보안 취약점 | 커밋 차단 → 즉시 수정 |
| 🟡 Warning | 성능 저하·규칙 위반·잠재 버그 | 커밋 가능 → 이번 작업 중 수정 |
| 🟢 Suggestion | 가독성·구조 개선 | 다음 작업 시 반영 |

## 자동 리뷰 흐름
```
git commit
  → pre-commit hook 자동 발동
  → Unity .cs 파일 감지
  → Critical 발견 → 커밋 차단
  → Critical 없음 → 커밋 통과 + docs/analysis/ 자동 생성
```

## 수동 리뷰
- `/review [파일]` — 특정 파일 즉시 리뷰
- `/review` — 최근 변경 파일 자동 탐지
- "이 코드 리뷰해줘" — 에이전트 자동 호출

## 긴급 커밋 (리뷰 생략)
```bash
git commit --no-verify -m "[hotfix] 긴급 수정"
```
