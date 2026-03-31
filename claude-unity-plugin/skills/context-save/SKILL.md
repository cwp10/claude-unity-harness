---
name: context-save
description: >
  Saves the current session's progress to claude-progress.txt and commits to git.
  Usage: /context-save
  Updates claude-progress.txt with what was done, what's next, and any issues.
  Commits all changes with a descriptive message.
  Run at the end of every work session.
allowed-tools: Read, Glob, Write, Bash
---

## 실행 순서

### 1단계: 현재 상태 수집

```bash
date '+%Y-%m-%d'
git log --oneline -5 2>/dev/null || echo "git 없음"
git diff --stat 2>/dev/null || echo ""
git status --short 2>/dev/null || echo ""
```

기존 `claude-progress.txt` 읽기 (있으면).

### 2단계: 이번 세션 작업 파악

현재 대화에서 추출:
- 완료된 작업
- 미완료·진행 중인 작업
- 내린 설계 결정과 이유
- 발견된 문제점과 임시 해결책
- 다음 세션에서 할 작업

### 3단계: claude-progress.txt 업데이트

기존 파일이 있으면 업데이트, 없으면 새로 생성:

```
# claude-progress.txt

## 프로젝트 정보
프로젝트: [프로젝트명]
타입: Unity
마지막 업데이트: YYYY-MM-DD

## 세션 이력
| 날짜 | 작업 내용 | 완료 여부 |
|------|----------|----------|
| YYYY-MM-DD | (이번 세션 작업 한 줄 요약) | ✅ / 🔄 |
| (이전 이력 유지) | | |

## 현재 진행 상황
(지금 이 시점 코드베이스 상태 2~3줄)

## 다음 작업
- [ ] (가장 중요한 것부터)
- [ ] ...

## 주요 설계 결정
- YYYY-MM-DD: (결정 내용 — 이유)
- (이전 결정 유지)

## 알려진 이슈
- (발견된 버그·제한사항)
- 없으면 "없음"
```

### 3.5단계: feature_list.json 업데이트

`feature_list.json` 파일 Read 후:
- 이번 세션에서 passes: true 로 변경된 항목 확인 및 저장
- 새로 추가된 기능 항목 확인
- 전체 진행률 계산 후 claude-progress.txt 에 반영:
  ```
  진행률: N/M 기능 완료 (passes: true)
  ```

### 4단계: git commit

변경사항이 있으면:
```bash
git add -A
git commit -m "작업: (이번 세션 핵심 변경 한 줄 요약)

- (세부 변경사항 1)
- (세부 변경사항 2)"
```

커밋 완료 후 "다음 세션에서 /context-load 로 복구하세요" 안내.
