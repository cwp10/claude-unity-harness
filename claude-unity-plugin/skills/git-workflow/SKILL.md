---
name: git-workflow
description: >
  Provides Git commit message conventions, branch strategy, and project-specific Git rules for Unity and web projects.
  Loads automatically when working with commits, branches, or Git operations.
  Do NOT use for non-Git version control or deployment workflows.
allowed-tools: Bash, Read
user-invocable: false
---

# Git 워크플로우 실행 가이드

## 브랜치 전략
```
main          ← 출시/배포 가능 상태만
develop       ← 통합 개발 브랜치
feature/xxx   ← 기능 개발
fix/xxx       ← 버그 수정
hotfix/xxx    ← 긴급 수정 (main에서 분기)
release/x.x.x ← 출시 준비
```

## 커밋 메시지 컨벤션
```
[타입] 간략 설명 (50자 이내)
```

| 타입 | 용도 |
|------|------|
| `[feat]` | 새 기능 |
| `[fix]` | 버그 수정 |
| `[refactor]` | 리팩토링 |
| `[perf]` | 성능 개선 |
| `[docs]` | 문서/주석 |
| `[chore]` | 빌드 설정, 패키지 |
| `[art]` | 에셋 추가/수정 (Unity) |

### Unity 예시
```
[feat] 플레이어 이중 점프 구현
[fix] 씬 전환 시 BGM 중복 재생 버그 수정
[art] 메인 캐릭터 idle 애니메이션 교체
[perf] 적 AI Update → 코루틴으로 최적화
```


## 필수 규칙

**필수 규칙:**
- `.meta` 파일 반드시 커밋 (누락 시 참조 깨짐)
- `Library/`, `Temp/`, `Builds/`, `Logs/` → `.gitignore`
- 대용량 에셋 (>100MB): Git LFS 사용
- `ProjectSettings/` 항상 커밋


## 커밋 단위 원칙
- 1커밋 = 1논리적 변경
- 너무 크면 `git add -p` 로 분할
- WIP 커밋 허용 → 나중에 squash
