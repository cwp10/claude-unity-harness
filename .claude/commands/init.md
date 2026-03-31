---
description: >
  Detects project type and updates .claude/CLAUDE.md with correct stack imports.
  Run once at the start of a new project to configure Claude for the project's stack.
  Supports Unity game, Unity industrial/education, Next.js web projects.
  In project install mode: updates the existing CLAUDE.md rather than creating a new one.
  Use /init --force to re-run on an already-initialized project.
allowed-tools: Read, Glob, Bash, Write
---

요청: $ARGUMENTS

## 실행 순서

### 1단계: 이미 초기화된 프로젝트 확인

`claude-progress.txt` 존재 여부 확인.

파일이 있고 `--force` 인자가 없으면:
```
"이미 초기화된 프로젝트입니다.
재초기화하려면 /init --force 를 실행하세요."
→ 종료
```

### 2단계: 프로젝트 타입 자동 감지

다음 순서로 탐색한다. (Glob, Bash 사용)

**Unity 프로젝트 감지:**
```
Glob: Assets/  폴더 존재 여부
Glob: **/*.cs  파일 존재 여부
Glob: ProjectSettings/ProjectVersion.txt 존재 여부
```

**웹 프로젝트 감지:**
```
Read: package.json → "next" 패키지 포함 여부
Glob: next.config.* 존재 여부
```

**감지 결과 분류:**
| 조건 | 타입 |
|------|------|
| Assets/ + .cs | unity |
| package.json next + next.config | web |
| 판단 불가 | 사용자에게 선택 요청 |

판단 불가 시:
```
"프로젝트 타입을 감지하지 못했습니다. 선택해주세요:
1. Unity 프로젝트
2. Next.js 웹
3. 직접 입력 (다른 스택)
번호를 입력해주세요:"
```

3번 선택 시 → 4단계에서 "custom" 타입으로 처리.

### 3단계: 프로젝트 이름 추출

```
현재 폴더명을 프로젝트 이름으로 사용
Bash: basename $(pwd)
```

### 4단계: CLAUDE.md 에 프로젝트 정보 추가

기존 `.claude/CLAUDE.md` 를 Read 한 후, 아래 섹션을 **파일 끝에 추가(append)** 한다.
기존 규칙 로드 섹션(`@rules/...`)은 그대로 유지한다.

---

**unity 타입 추가 내용:**

```markdown
## 스택 로드
@engines/unity.md
@languages/csharp.md
@domains/unity.md

## 프로젝트 정보
- 프로젝트명: [프로젝트명]
- Unity 버전: (ProjectSettings/ProjectVersion.txt 에서 읽어서 채움)
- 타겟 플랫폼: (미정 — 수정 필요)
- Render Pipeline: (미정 — 수정 필요)
- 프로젝트 성격: (게임 / 산업 시뮬레이션 / 교육 콘텐츠 / 혼합 — 수정 필요)
- 납품처: (해당 시 입력)

## Hook 설정
# Unity Hub 기본 경로로 자동 탐색됩니다.
# 커스텀 설치 경로라면 아래 주석을 해제하고 경로를 입력하세요:
# UNITY_PATH: C:/Program Files/Unity/Hub/Editor/[버전]/Editor/Unity.exe

## 폴더 구조
(Assets/_Project/Scripts/ 구조를 Glob으로 읽어서 채움)

## 이 프로젝트만의 특이사항
- (직접 입력)

## 문서 카테고리 맵
# /analyze 커맨드가 문서 병합 대상을 결정할 때 사용
# 예시 (프로젝트에 맞게 수정):
# - 전투-시스템: Combat/, Damage/, StatusEffect/
# - 인벤토리: Inventory/, Item/, Shop/
# - 플레이어: Player/, Input/, Camera/
```

---

**custom 타입 추가 내용:**

@import 없이 사용 가능한 모든 스택을 주석으로 나열.

```markdown
## 스택 로드
# 필요한 줄의 # 을 지워서 활성화하세요
# @engines/unity.md
# @languages/csharp.md
# @languages/typescript.md
# @domains/unity.md
# @domains/web.md

## 프로젝트 정보
- 프로젝트명: [프로젝트명]
- 언어/프레임워크: (직접 입력)
- 타겟: (직접 입력)
- 빌드 명령어: (직접 입력)

## 이 프로젝트만의 특이사항
- (직접 입력)

## 문서 카테고리 맵
# 예시:
# - 기능명: 관련폴더1/, 관련폴더2/
```

---

**web 타입 추가 내용:**

```markdown
## 스택 로드
@languages/typescript.md
@domains/web.md

## 프로젝트 정보
- 프로젝트명: [프로젝트명]
- Framework: Next.js 15 (App Router + Turbopack)
- Runtime: React 19 + TypeScript 5
- Styling: TailwindCSS v4 + shadcn/ui (new-york style)
- Forms: React Hook Form + Zod + Server Actions
- 배포: (미정 — 수정 필요)
- 프로젝트 성격: (SaaS / 관리도구 / 교육플랫폼 — 수정 필요)

## 폴더 구조
(src/ 또는 app/ 구조를 Glob으로 읽어서 채움)

## 이 프로젝트만의 특이사항
- (직접 입력)

## 문서 카테고리 맵
# 예시:
# - 인증: auth/, middleware, session
# - 사용자관리: users/, profile/
```

### 5단계: pre-commit hook 확인

`.git` 폴더 존재 여부 확인.

```
.git 폴더 있음
  .git/hooks/pre-commit 이미 존재 → "hook 이미 설치됨" 출력
  없으면 → .claude/hooks/pre-commit 을 .git/hooks/pre-commit 으로 복사
           chmod +x .git/hooks/pre-commit
           "pre-commit hook 설치 완료" 출력

.git 폴더 없음
  → hook 설치 건너뜀
  → "git init 후 아래 명령어로 hook을 설치하세요" 안내 출력
```

Windows 환경에서 Bash 복사가 실패하면 PowerShell 명령어로 안내:
```
powershell -ExecutionPolicy Bypass -File ".\.claude\hooks\install-hooks.ps1"
```

### 5.5단계: 하네스 파일 생성

`.git` 폴더 있는 경우 (git 프로젝트일 때만) 아래 두 파일을 프로젝트 루트에 생성한다.

**파일 1: claude-progress.txt**

```
# claude-progress.txt

## 프로젝트 정보
프로젝트: [프로젝트명]
타입: [Unity / 웹]
초기화: YYYY-MM-DD

## 세션 이력
| 날짜 | 작업 내용 | 완료 여부 |
|------|----------|----------|
| YYYY-MM-DD | 프로젝트 초기화 | ✅ |

## 현재 진행 상황
초기화 완료. feature_list.json 의 기능 목록을 확인하고 첫 번째 기능부터 시작하세요.

## 다음 작업
feature_list.json 참조

## 알려진 이슈
없음
```

**파일 2: feature_list.json**

프로젝트 타입에 따라 기본 기능 목록을 생성한다.
각 기능은 `passes: false` 로 시작하고, 구현 후 검증이 완료되면 `passes: true` 로 변경한다.

Unity 기본 템플릿:
```json
[
  {
    "id": "F001",
    "category": "core",
    "description": "씬 로드 및 기본 화면 진입",
    "passes": false
  },
  {
    "id": "F002",
    "category": "core",
    "description": "플레이어 입력 처리",
    "passes": false
  }
]
```

웹 기본 템플릿:
```json
[
  {
    "id": "F001",
    "category": "core",
    "description": "메인 페이지 렌더링",
    "passes": false
  },
  {
    "id": "F002",
    "category": "core",
    "description": "기본 라우팅 동작",
    "passes": false
  }
]
```

**중요 규칙:**
- 기능 항목을 삭제하거나 description을 수정하지 않는다
- `passes: true` 변경은 실제 동작 검증 후에만 한다
- 새 기능 추가는 목록 끝에 추가한다

이 두 파일은 세션 간 컨텍스트를 유지하는 핵심 하네스 파일이다.
매 작업 세션 종료 시 `/context-save` 로 claude-progress.txt 를 업데이트한다.

### 6단계: 결과 출력

```
## ✅ 초기화 완료

프로젝트: [프로젝트명]
타입:     [감지된 타입]
설정파일: .claude/CLAUDE.md (프로젝트 정보 추가됨)

로드된 스택:
  - [해당 스택 목록]

pre-commit hook:
  [설치 완료 / 이미 설치됨 / .git 없음 - git init 후 수동 설치 필요]

다음 단계:
1. .claude/CLAUDE.md 열어서 "수정 필요" 항목 채우기
2. 문서 카테고리 맵 프로젝트에 맞게 수정
3. claude-progress.txt 에 첫 번째 작업 목록 추가
```
