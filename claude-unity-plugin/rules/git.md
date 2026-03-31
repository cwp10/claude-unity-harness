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
[feat]     새 기능
[fix]      버그 수정
[refactor] 리팩토링
[perf]     성능 개선
[docs]     문서·주석
[art]      에셋 추가·수정 (Unity)
[chore]    설정·의존성
```
커밋은 하나의 논리적 변경 단위. 너무 크면 `git add -p` 로 분할.

### Unity 커밋 예시
```
[feat] 플레이어 이중 점프 구현
[fix]  씬 전환 시 BGM 중복 재생 버그 수정
[art]  메인 캐릭터 idle 애니메이션 교체
[perf] 적 AI Update → 코루틴으로 최적화
```

## 절대 금지
- main 브랜치 직접 push
- 빌드 안 되는 상태로 커밋
- 민감 정보(API 키, 비밀번호) 커밋
- .meta 파일 누락 (Unity)
