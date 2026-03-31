---
name: context-load
description: >
  Loads session context by reading claude-progress.txt and git history.
  Usage: /context-load
  Runs the standard session startup routine: read progress file, check git log,
  verify current state, then present what to work on next.
  Run at the start of every work session to get up to speed.
allowed-tools: Read, Glob, Bash
---

## 세션 시작 루틴 (공식 하네스 패턴)

### 1단계: 현재 위치 확인
```bash
pwd
```

### 2단계: claude-progress.txt 읽기

`claude-progress.txt` 파일 Read.

파일 없음 → "claude-progress.txt 가 없습니다. /setup 으로 프로젝트를 초기화하세요." 출력 후 종료.

### 2.5단계: feature_list.json 읽기

`feature_list.json` 파일 Read (있으면).
passes: false 인 항목 수와 다음 작업할 기능 확인.

### 3단계: git 이력 확인
```bash
git log --oneline -10 2>/dev/null || echo "git 이력 없음"
```

### 4단계: 현재 상태 출력

```
## 세션 복구 완료

### 마지막 작업
(claude-progress.txt 의 세션 이력 최근 항목)

### 현재 진행 상황
(claude-progress.txt 의 현재 진행 상황)

### 최근 커밋
(git log 결과)

### 다음 작업
(feature_list.json 의 passes: false 중 최우선 항목)

### 알려진 이슈
(claude-progress.txt 의 알려진 이슈)
```

### 5단계: 작업 재개 제안

"1번 작업부터 시작할까요?" 또는 "어떤 작업을 시작할까요?" 질문.
