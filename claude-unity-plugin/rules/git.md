# Git 워크플로우 규칙

## 브랜치 전략 (1인 개발 기준)
- `main` — 출시/배포 가능 상태만
- `develop` — 통합 개발 브랜치
- `feature/xxx` — 기능 개발
- `fix/xxx` — 버그 수정
- `hotfix/xxx` — 긴급 수정 (main에서 분기)
- `release/x.x.x` — 출시 준비

## 커밋 메시지 컨벤션
```
[feat]    새 기능
[fix]     버그 수정
[refactor] 리팩토링
[docs]    문서
[chore]   설정·의존성
```
커밋은 하나의 논리적 변경 단위. 너무 크면 나눌 것.

## 절대 금지
- main 브랜치 직접 push
- 빌드 안 되는 상태로 커밋
- 민감 정보(API 키, 비밀번호) 커밋
- .meta 파일 누락 (Unity)
