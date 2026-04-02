---
name: context-save
description: >
  Saves the current session's progress to .claude/claude-progress.txt and .claude/project-memory.json, then optionally commits to git.
  Usage: /context-save
  Updates the human-readable progress log, structured project memory JSON, and feature_list.json.
  Asks for confirmation before committing to git.
  Run at the end of every work session.
tools: Read, Glob, Write, Bash, mcp__UnityMCP__run_tests, mcp__UnityMCP__read_console
model: haiku
triggers:
  - "세션 종료"
  - "작업 저장"
  - "오늘 마무리"
  - "오늘 작업 끝"
  - "마무리하고 종료"
keywords: [save, commit, session, progress, end]
---

## 실행 순서

### 0단계: 절대 경로 확보 (필수)

**파일 쓰기 전에 반드시 먼저 실행한다.**

```bash
git rev-parse --show-toplevel 2>/dev/null || pwd
```

출력된 경로를 `PROJECT_ROOT` 로 기억한다.
이후 모든 파일 쓰기는 반드시 절대 경로를 사용한다:
- `<PROJECT_ROOT>/.claude/claude-progress.txt`
- `<PROJECT_ROOT>/.claude/project-memory.json`
- `<PROJECT_ROOT>/.claude/feature_list.json`

> ⚠️ 절대 경로 없이 `.claude/...` 상대 경로로 Write를 호출하면 반드시 실패한다.

### 1단계: 현재 상태 수집

```bash
date '+%Y-%m-%d'
git log --oneline -5 2>/dev/null || echo "git 없음"
git diff --stat 2>/dev/null || echo ""
git status --short 2>/dev/null || echo ""
```

기존 `<PROJECT_ROOT>/.claude/claude-progress.txt` 읽기 (있으면).

### 2단계: 이번 세션 작업 파악

현재 대화에서 추출:
- 완료된 작업
- 미완료·진행 중인 작업
- 내린 설계 결정과 이유
- 발견된 문제점과 임시 해결책
- 다음 세션에서 할 작업

### 2.5단계: MCP 테스트 실행 (Unity Editor 연결 시)

UnityMCP 툴 사용 가능 여부에 따라 분기한다.

**연결됨 → 테스트 실행 후 커밋:**

1. `run_tests` 툴 호출 — EditMode 테스트 전체 실행
2. `read_console` 툴 호출 — 에러 로그 확인
3. 결과 출력:
   - 전체 통과: `✅ 테스트 통과 (N/N)` → 4단계(커밋)로 진행
   - 실패 있음: 실패 목록 출력 → "테스트 실패가 있습니다. 그래도 커밋할까요?" 질문
   - 사용자 확인 후 진행

**미연결 → 테스트 생략:**

- `[MCP 미연결] 테스트 생략 — 커밋을 진행합니다.` 출력 후 3단계로 진행

### 3단계: .claude/claude-progress.txt 업데이트

기존 파일이 있으면 업데이트, 없으면 새로 생성:

```
# .claude/claude-progress.txt

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

### 3.5단계: .claude/project-memory.json 업데이트

`.claude/project-memory.json` 파일을 생성하거나 업데이트한다:

```json
{
  "techStack": "[Unity 버전, C# 버전, Render Pipeline]",
  "platform": "[Android / iOS / PC / WebGL]",
  "currentFeature": "[지금 작업 중인 기능명]",
  "phase": "[설계 중 / 구현 중 / 검증 중 / 완료]",
  "conventions": {
    "naming": "m_(private) k_(const) s_(static)",
    "async": "UniTask",
    "assets": "Addressables",
    "events": "ScriptableObject Event Channel"
  },
  "decisions": [
    { "date": "YYYY-MM-DD", "decision": "[결정 내용]", "reason": "[이유]" }
  ],
  "blockers": ["[현재 막혀 있는 문제 — 없으면 빈 배열]"],
  "nextUp": "[다음 작업할 기능명]",
  "progress": "N/M 기능 완료",
  "lastUpdated": "YYYY-MM-DD"
}
```

기존 파일이 있으면 필드를 병합 업데이트 (decisions 배열은 append).
ProjectSettings/ProjectVersion.txt 에서 Unity 버전 읽어 techStack 채우기.

### 3.6단계: .claude/feature_list.json 업데이트

`.claude/feature_list.json` 파일 Read 후:
- 이번 세션에서 passes: true 로 변경된 항목 확인 및 저장
- 새로 추가된 기능 항목 확인
- 전체 진행률 계산 후 .claude/claude-progress.txt 에 반영:
  ```
  진행률: N/M 기능 완료 (passes: true)
  ```

### 4단계: git commit (확인 후 실행)

**git 사용 가능 여부 먼저 확인:**

```bash
git rev-parse --git-dir 2>/dev/null
```

- 실패(git 저장소 아님) → "git 저장소가 아닙니다. 파일 저장만 완료되었습니다." 출력 후 종료
- 성공 → 아래 확인 진행

파일 저장 완료 후 아래와 같이 확인한다:

```
✅ 세션 저장 완료
- .claude/claude-progress.txt 업데이트
- .claude/project-memory.json 업데이트
- .claude/feature_list.json 진행률 반영

커밋할까요? (y/N)
```

**승인 시** — 이번 세션의 주요 변경 성격에 따라 커밋 타입을 선택해 실행:

| 세션 내용 | 커밋 타입 |
|----------|----------|
| 새 기능 구현 완료 | `[feat]` |
| 버그 수정 | `[fix]` |
| 리팩토링 | `[refactor]` |
| 문서·분석 파일만 | `[docs]` |
| 설정·하네스 파일 | `[chore]` |

```bash
git add -A
git commit -m "[타입] (이번 세션 핵심 변경 한 줄 요약)

- (세부 변경사항 1)
- (세부 변경사항 2)"
```

**거절 시** — "다음 세션에서 /context-load 로 복구하세요." 안내 후 종료.
