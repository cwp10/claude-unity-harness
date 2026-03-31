---
name: setup-check
description: >
  플러그인 설치 상태와 프로젝트 초기화 상태를 진단한다.
  Usage: /setup-check
  pre-commit hook, CLAUDE.md 설정, 하네스 파일, Unity 폴더 구조를 점검한다.
  문제 항목은 수정 방법과 함께 출력한다.
allowed-tools: Read, Glob, Bash
cost: haiku
triggers:
  - "설치 확인"
  - "세팅 확인"
  - "환경 점검"
  - "setup 확인"
  - "정상 동작해?"
keywords: [setup, check, doctor, diagnose, install, health]
---

## 실행 순서

다음 항목을 순서대로 확인하고 결과를 표로 출력한다.

### 1. pre-commit hook
```bash
[ -x .git/hooks/pre-commit ] && echo "OK" || echo "MISSING"
```

### 2. .claude/CLAUDE.md 설정
- 파일 존재 여부
- `@engines/unity.md` 활성화 여부 (주석 처리 안 됐는지)
- `@languages/csharp.md` 활성화 여부

### 3. 하네스 파일
```bash
[ -f feature_list.json ] && echo "OK" || echo "MISSING"
[ -f .claude/project-memory.json ] && echo "OK" || echo "MISSING"
[ -f claude-progress.txt ] && echo "OK" || echo "MISSING"
```

### 4. Unity 프로젝트 구조
```bash
[ -d Assets/_Project/Scripts ] && echo "OK" || echo "MISSING"
[ -f ProjectSettings/ProjectVersion.txt ] && echo "OK" || echo "MISSING"
```

### 5. Unity 버전 확인
```bash
cat ProjectSettings/ProjectVersion.txt 2>/dev/null | grep "m_EditorVersion" || echo "확인 불가"
```

### 6. .gitignore 확인
- `Library/` 제외 여부
- `Temp/` 제외 여부

---

## 출력 형식

```
## 설치 상태 진단

| 항목 | 상태 | 비고 |
|------|------|------|
| pre-commit hook | ✅/❌ | |
| CLAUDE.md 스택 규칙 | ✅/⚠️ | engines/unity.md 활성화 여부 |
| feature_list.json | ✅/❌ | |
| project-memory.json | ✅/⚠️ | 없으면 /context-save 로 생성 |
| claude-progress.txt | ✅/⚠️ | |
| Unity 폴더 구조 | ✅/⚠️ | Assets/_Project/Scripts/ |
| Unity 버전 | ✅/⚠️ | [버전] |
| .gitignore | ✅/⚠️ | Library/ 제외 여부 |

### 문제 항목 수정 방법
(❌/⚠️ 항목만 출력)
- ❌ pre-commit hook: `/setup` 재실행 또는 `cp .claude/hooks/pre-commit .git/hooks/pre-commit && chmod +x .git/hooks/pre-commit`
- ⚠️ project-memory.json: `/context-save` 실행으로 자동 생성
```
