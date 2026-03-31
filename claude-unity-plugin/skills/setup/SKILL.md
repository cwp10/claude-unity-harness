---
name: setup
description: >
  Unity 프로젝트를 감지하고 .claude/CLAUDE.md에 스택 정보를 추가한다.
  하네스 파일(claude-progress.txt, feature_list.json)을 생성하고 pre-commit hook을 설치한다.
  프로젝트 루트에서 최초 1회 실행. 재실행은 /setup --force.
allowed-tools: Read, Glob, Bash, Write
cost: sonnet
triggers:
  - "처음 설정"
  - "프로젝트 초기화"
  - "setup 해줘"
keywords: [setup, init, project, unity, harness]
---

요청: $ARGUMENTS

## 실행 순서

### 1단계: 초기화 여부 확인

`claude-progress.txt` 존재 여부 확인 (Read 시도).

파일이 있고 `--force` 인자가 없으면:
```
"이미 초기화된 프로젝트입니다.
재초기화하려면 /setup --force 를 실행하세요."
→ 종료
```

### 2단계: Unity 프로젝트 감지

아래 3가지를 Glob으로 확인한다:
```
Assets/          폴더 존재 여부
**/*.cs          파일 존재 여부
ProjectSettings/ProjectVersion.txt  존재 여부
```

3가지 모두 없으면:
```
"Unity 프로젝트를 감지하지 못했습니다.
Assets/ 폴더와 ProjectSettings/ 폴더가 있는
Unity 프로젝트 루트에서 실행하세요.
계속 진행하시겠습니까? (y/N)"
```
→ y 면 계속, N 이면 종료.

### 3단계: 프로젝트 기본 정보 수집

아래 2가지를 실제로 읽어서 값을 확보한다.

**Unity 버전 추출:**
```bash
# ProjectSettings/ProjectVersion.txt 를 Read하여 m_EditorVersion 값 추출
```
읽기 실패 시 → `(확인 필요)` 로 대체.

**폴더 구조 파악:**
```
Glob: Assets/_Project/Scripts/**  (2~3단계 깊이)
없으면: Glob: Assets/**/*.cs 로 상위 폴더 파악
```
실제 폴더 트리를 3~5줄로 요약한다.

**프로젝트명:**
```bash
basename $(pwd)
```

### 4단계: CLAUDE.md 업데이트

#### --force 없는 최초 실행:
기존 `.claude/CLAUDE.md` 를 Read 한 후 파일 **끝에 아래 내용을 추가(append)** 한다.

#### --force 재실행:
기존 CLAUDE.md에서 `## 스택 로드` 섹션부터 `## 문서 카테고리 맵` 섹션 끝까지를
아래 내용으로 **교체(replace)** 한다. (중복 방지)

---

**추가/교체 내용 (3단계에서 수집한 실제 값으로 채워서 작성):**

```markdown
## 스택 로드
@engines/unity.md
@languages/csharp.md
@domains/unity.md

## 프로젝트 정보
- 프로젝트명: [3단계에서 추출한 이름]
- Unity 버전: [3단계에서 읽은 실제 버전]
- 타겟 플랫폼: (수정 필요 — PC / Android / iOS / WebGL 등)
- Render Pipeline: (수정 필요 — URP / HDRP / Built-in)
- 프로젝트 성격: (수정 필요 — 게임 / 산업 시뮬레이션 / 교육 콘텐츠 / 혼합)
- 납품처: (해당 시 입력)

## Hook 설정
# Unity Hub 기본 경로로 자동 탐색됩니다.
# 커스텀 경로라면 아래 주석을 해제하고 입력하세요:
# UNITY_PATH: C:/Program Files/Unity/Hub/Editor/[버전]/Editor/Unity.exe

## 폴더 구조
[3단계에서 파악한 실제 폴더 구조 요약]

## 이 프로젝트만의 특이사항
- (직접 입력)

## 문서 카테고리 맵
# /analyze 커맨드가 문서 저장 위치를 결정할 때 참조
# 아래 예시를 프로젝트에 맞게 수정하세요:
# - 전투-시스템: Combat/, Damage/, StatusEffect/
# - 인벤토리: Inventory/, Item/, Shop/
# - 플레이어: Player/, Input/, Camera/
# - 시뮬레이션: Simulation/, Physics/, Machine/
# - 학습흐름: Learning/, Step/, Quiz/
```

---

### 5단계: 하네스 파일 생성

`.git` 여부와 관계없이 항상 아래 두 파일을 프로젝트 루트에 생성한다.
(--force 재실행 시에도 덮어쓴다)

**파일 1: claude-progress.txt**

```
# claude-progress.txt

## 프로젝트 정보
프로젝트: [3단계 프로젝트명]
타입: Unity
초기화: [오늘 날짜 YYYY-MM-DD]

## 세션 이력
| 날짜 | 작업 내용 | 완료 여부 |
|------|----------|----------|
| [오늘 날짜] | 프로젝트 초기화 | ✅ |

## 현재 진행 상황
초기화 완료. feature_list.json 의 기능 목록을 확인하고 첫 번째 기능부터 시작하세요.

## 다음 작업
feature_list.json 참조

## 알려진 이슈
없음
```

**파일 2: feature_list.json**

CLAUDE.md의 "프로젝트 성격"이 이미 채워져 있으면 그에 맞는 템플릿,
아직 미정이면 게임 기본 템플릿으로 생성한다.

게임:
```json
[
  { "id": "F001", "category": "core", "description": "씬 로드 및 기본 화면 진입", "passes": false },
  { "id": "F002", "category": "core", "description": "플레이어 입력 처리", "passes": false },
  { "id": "F003", "category": "core", "description": "카메라 제어", "passes": false }
]
```

산업 시뮬레이션:
```json
[
  { "id": "F001", "category": "core", "description": "시뮬레이션 씬 로드 및 초기화", "passes": false },
  { "id": "F002", "category": "core", "description": "장비·오브젝트 인터랙션", "passes": false },
  { "id": "F003", "category": "core", "description": "시뮬레이션 상태 저장/불러오기", "passes": false }
]
```

교육 콘텐츠:
```json
[
  { "id": "F001", "category": "core", "description": "학습 씬 로드 및 진입", "passes": false },
  { "id": "F002", "category": "core", "description": "학습 단계 진행 흐름", "passes": false },
  { "id": "F003", "category": "core", "description": "퀴즈·평가 결과 저장", "passes": false }
]
```

**중요 규칙:**
- 기능 항목 삭제·description 수정 금지
- `passes: true` 변경은 실제 검증 후에만
- 새 기능 추가는 목록 끝에 추가

### 6단계: pre-commit hook 설치

`.git/hooks/pre-commit` 존재 여부 확인.

```
이미 존재 → "hook 이미 설치됨" 출력, 건너뜀
없음 + .git 있음 → 아래 스크립트 내용을 Write 툴로 .git/hooks/pre-commit 에 직접 작성
                   chmod +x .git/hooks/pre-commit (Bash 툴)
                   "pre-commit hook 설치 완료" 출력
.git 없음 → 건너뜀. 안내:
            git init 후 /setup --force 재실행
```

**.git/hooks/pre-commit 에 Write 할 내용:**

```sh
#!/bin/sh
# Claude Code 코드 리뷰 + 자동 문서화 pre-commit hook (Unity 전용)
# Git for Windows (Git Bash) 환경에서 동작

RED='\033[0;31m'
YELLOW='\033[1;33m'
GREEN='\033[0;32m'
CYAN='\033[0;36m'
BLUE='\033[0;34m'
NC='\033[0m'

CLAUDE_CMD=""
if command -v claude > /dev/null 2>&1; then
    CLAUDE_CMD="claude"
elif [ -f "$APPDATA/npm/claude.cmd" ]; then
    CLAUDE_CMD="$APPDATA/npm/claude.cmd"
fi

CHANGED_CS=$(git diff --cached --name-only --diff-filter=ACM | grep '\.cs$' || true)

if [ -z "$CHANGED_CS" ]; then
    printf "${GREEN}리뷰 대상 파일 없음 (.cs 파일 변경 없음).${NC}\n\n"
    exit 0
fi

# ── STEP 0: .meta 파일 처리 ─────────────────────────────────
printf "\n${CYAN}[0/3] .meta 파일 확인${NC}\n\n"
STAGED_ASSETS=$(git diff --cached --name-only --diff-filter=ACM \
    | grep -E '\.(cs|prefab|asset|unity|mat|anim|controller|png|jpg|fbx)$' || true)
MISSING_META=""
if [ -n "$STAGED_ASSETS" ]; then
    while IFS= read -r asset; do
        META="${asset}.meta"
        if [ -f "$META" ]; then
            if ! git diff --cached --name-only | grep -qF "$META"; then
                git add "$META"
            fi
        else
            MISSING_META="${MISSING_META}  ! ${META}\n"
        fi
    done << ASSET_EOF
$STAGED_ASSETS
ASSET_EOF
fi
if [ -n "$MISSING_META" ]; then
    printf "${RED}커밋 차단: .meta 파일 누락${NC}\n"
    printf "$MISSING_META\n"
    printf "Unity 에디터에서 Ctrl+S 후 다시 커밋하세요.\n"
    exit 1
fi
printf "${GREEN}.meta 파일 확인 완료.${NC}\n\n"

# ── STEP 1: Unity 컴파일 검증 ───────────────────────────────
printf "${CYAN}[1/3] Unity 컴파일 검증${NC}\n\n"
UNITY_VERSION=""
UNITY_EXE=""
VERSION_FILE="ProjectSettings/ProjectVersion.txt"
if [ -f "$VERSION_FILE" ]; then
    UNITY_VERSION=$(grep "m_EditorVersion:" "$VERSION_FILE" | awk '{print $2}' | tr -d '\r')
    if [ -f ".claude/CLAUDE.md" ]; then
        CUSTOM_PATH=$(grep "UNITY_PATH:" ".claude/CLAUDE.md" | sed 's/.*UNITY_PATH: *//' | tr -d '\r')
        [ -n "$CUSTOM_PATH" ] && UNITY_EXE=$(echo "$CUSTOM_PATH" | sed 's|\\|/|g' | sed 's|^\([A-Za-z]\):|/\L\1|')
    fi
    if [ -z "$UNITY_EXE" ] || [ ! -f "$UNITY_EXE" ]; then
        UNITY_EXE_C="/c/Program Files/Unity/Hub/Editor/${UNITY_VERSION}/Editor/Unity.exe"
        UNITY_EXE_D="/d/Program Files/Unity/Hub/Editor/${UNITY_VERSION}/Editor/Unity.exe"
        [ -f "$UNITY_EXE_C" ] && UNITY_EXE="$UNITY_EXE_C" || { [ -f "$UNITY_EXE_D" ] && UNITY_EXE="$UNITY_EXE_D"; }
    fi
fi
if [ -z "$UNITY_VERSION" ]; then
    printf "${YELLOW}ProjectVersion.txt 없음. 컴파일 검증 생략.${NC}\n\n"
elif [ -z "$UNITY_EXE" ] || [ ! -f "$UNITY_EXE" ]; then
    printf "${YELLOW}Unity.exe 경로 없음. 컴파일 검증 생략.${NC}\n"
    printf ".claude/CLAUDE.md 에 UNITY_PATH: 추가 후 재시도.\n\n"
else
    COMPILE_LOG=".claude/compile_check.log"
    mkdir -p ".claude"
    COMPILE_EXIT=0
    timeout 60 "$UNITY_EXE" -batchmode -quit -projectPath "$(pwd)" -logFile "$COMPILE_LOG" > /dev/null 2>&1 || COMPILE_EXIT=$?
    COMPILE_ERRORS=""
    [ -f "$COMPILE_LOG" ] && COMPILE_ERRORS=$(grep -iE "^.*error (CS|Unity)[0-9]+.*$" "$COMPILE_LOG" || true)
    if [ -n "$COMPILE_ERRORS" ] || [ "$COMPILE_EXIT" -eq 124 ]; then
        printf "${RED}커밋 차단: 컴파일 에러${NC}\n\n"
        [ -n "$COMPILE_ERRORS" ] && echo "$COMPILE_ERRORS"
        exit 1
    fi
    printf "${GREEN}컴파일 성공.${NC}\n"
    rm -f "$COMPILE_LOG"
    printf "\n"
fi

# ── STEP 2: 코드 리뷰 ──────────────────────────────────────
FILE_COUNT=$(echo "$CHANGED_CS" | wc -l | tr -d ' ')
printf "${CYAN}[2/3] 코드 리뷰 (%d개 파일)${NC}\n\n" "$FILE_COUNT"
CRITICAL_COUNT=0
REVIEW_FILE=""
if [ -z "$CLAUDE_CMD" ]; then
    printf "${YELLOW}claude CLI 없음. 리뷰 생략.${NC}\n\n"
else
    REVIEW_OUTPUT=$($CLAUDE_CMD -p --bare \
"이번 커밋에서 변경된 Unity C# 파일들을 리뷰해줘.
변경 파일: $CHANGED_CS
이슈 분류: [Critical] 빌드실패·크래시·메모리릭 / [Warning] 성능저하·규칙위반 / [Suggestion] 가독성개선
Critical 이슈가 있으면 '커밋 차단'을 명시해줘." 2>/dev/null) || REVIEW_OUTPUT=""

    if [ -n "$REVIEW_OUTPUT" ]; then
        CRITICAL_COUNT=$(echo "$REVIEW_OUTPUT" | grep -c "\[Critical\]" || true)
        REVIEW_DIR="docs/analysis"
        mkdir -p "$REVIEW_DIR"
        REVIEW_FILE="${REVIEW_DIR}/review_$(date +%Y%m%d_%H%M%S).md"
        printf "# 커밋 전 코드 리뷰\n날짜: %s\n\n%s\n" "$(date '+%Y-%m-%d %H:%M:%S')" "$REVIEW_OUTPUT" > "$REVIEW_FILE"
        printf "%s\n\n" "$REVIEW_OUTPUT"
    fi
fi

if [ "$CRITICAL_COUNT" -gt 0 ]; then
    printf "${RED}커밋 차단: Critical 이슈 %d건${NC}\n" "$CRITICAL_COUNT"
    [ -n "$REVIEW_FILE" ] && printf "리뷰 내용: %s\n" "$REVIEW_FILE"
    printf "긴급 시: git commit --no-verify\n\n"
    exit 1
fi
printf "${GREEN}Critical 이슈 없음.${NC}\n\n"

# ── STEP 3: 문서 자동화 ────────────────────────────────────
printf "${CYAN}[3/3] 문서 자동화${NC}\n\n"
if [ -n "$CLAUDE_CMD" ]; then
    mkdir -p "docs/analysis"
    COMMIT_DATE=$(date '+%Y-%m-%d')
    PROCESSED_DIRS=""
    while IFS= read -r FILE; do
        [ -f "$FILE" ] || continue
        FILE_DIR=$(dirname "$FILE")
        case "$PROCESSED_DIRS" in *"|${FILE_DIR}|"*) continue ;; esac
        PROCESSED_DIRS="${PROCESSED_DIRS}|${FILE_DIR}|"
        DOC_RESULT=$($CLAUDE_CMD -p --bare \
"Unity C# 파일 분석 문서 작성: $FILE_DIR 의 변경 파일들
첫 줄 반드시: FILENAME: 한국어-기능명-분석.md
이후 마크다운: ## 개요 / ## 관련 파일 / ## 동작 흐름 / ## 주의사항 / ## 변경 이력 (날짜: $COMMIT_DATE)" 2>/dev/null) || continue
        DOC_FILENAME=$(echo "$DOC_RESULT" | head -1 | sed 's/FILENAME: //' | tr -d '\r')
        DOC_CONTENT=$(echo "$DOC_RESULT" | tail -n +2)
        [ -z "$DOC_FILENAME" ] && continue
        DOC_PATH="docs/analysis/${DOC_FILENAME}"
        printf "# %s\n\n작성일: %s\n\n%s\n" "$(echo "$DOC_FILENAME" | sed 's/\.md$//' | sed 's/-/ /g')" "$COMMIT_DATE" "$DOC_CONTENT" > "$DOC_PATH"
        git add "$DOC_PATH"
        printf "  ${BLUE}문서: %s${NC}\n" "$DOC_PATH"
    done << EOF
$CHANGED_CS
EOF
fi

printf "\n${GREEN}커밋을 진행합니다.${NC}\n\n"
exit 0
```

### 7단계: 결과 출력

```
## ✅ 초기화 완료

프로젝트: [프로젝트명]
Unity:    [버전]
설정파일: .claude/CLAUDE.md

로드된 스택:
  @engines/unity.md
  @languages/csharp.md
  @domains/unity.md

생성된 파일:
  claude-progress.txt
  feature_list.json

pre-commit hook: [설치 완료 / 이미 설치됨 / .git 없음]

다음 단계:
1. .claude/CLAUDE.md 에서 "수정 필요" 항목 채우기
2. 문서 카테고리 맵을 프로젝트에 맞게 수정
3. /context-load 로 세션 시작
```
