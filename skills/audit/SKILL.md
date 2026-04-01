---
name: audit
description: >
  Runs Unity code quality audit + build checks automatically.
  Usage: /audit                  ← 코드 품질 + 빌드 체크 (기본)
  Usage: /audit migrate          ← 구형 Unity API 탐지
  Usage: /audit localize         ← 하드코딩 문자열 탐지
  Unity: code quality audit (unity-reviewer) + .meta files, compile errors, folder structure.
  Both tasks run concurrently.
tools: Read, Glob, Grep, Bash, Write
model: sonnet
triggers:
  - "전체 감사"
  - "빌드 체크"
  - "전체 리뷰"
  - "프로젝트 점검"
  - "배포 전 확인"
  - "구형 API"
  - "마이그레이션"
  - "하드코딩"
  - "현지화"
  - "다국어 준비"
keywords: [audit, build, quality, meta, check, ci, migrate, localize]
---

서브커맨드: $ARGUMENTS

서브커맨드가 있으면 해당 모드로 실행한다:
- `migrate` → [migrate 모드](#migrate-모드)로 점프
- `localize` → [localize 모드](#localize-모드)로 점프
- 없음 → 기본 감사 모드 실행

## 실행 순서

### 1단계: 코드 품질 감사 + 빌드 체크 (concurrently)

아래 2개 작업을 동시에(concurrently) 실행한다:

**Task A — 코드 품질 감사 (unity-reviewer 에이전트):**

대상 결정:
`git diff --name-only HEAD` 로 최근 변경 파일 확인.
변경 파일 없으면 `Assets/_Project/Scripts/` 전체 .cs 파일 대상.

unity-reviewer 에이전트에게 위임:
- 감사 대상 파일 목록
- "전체 감사 모드 — 아래 4개 관점을 동시에(concurrently) 분석해서 결과 출력"
  - 성능 (Performance)
  - Unity 규칙
  - 아키텍처
  - 안전성
- Critical 0건이어도 전체 통계 출력

**Task B — 빌드 체크 (직접 실행):**

① .meta 파일 누락 검사:
```bash
find Assets -type f ! -name "*.meta" | while read f; do
  [ ! -f "$f.meta" ] && echo "MISSING META: $f"
done
```

② C# 컴파일 에러 탐지:
```bash
grep -rn "CS[0-9]\{4\}" . --include="*.cs" 2>/dev/null | head -20
```

③ 필수 폴더 구조 확인:
- `Assets/_Project/Scripts/` 존재 여부
- `Assets/_Project/Scenes/` 존재 여부

④ .gitignore 확인:
- `Library/`, `Temp/`, `obj/`, `Build/` 제외 여부

### 2단계: 결과 출력

```
## Unity 감사 결과

### 코드 품질
🔴 Critical N건 / 🟡 Warning N건 / 🟢 Suggestion N건
[이슈 상세]

### 빌드 체크
.meta 누락: 🔴 N건 / ✅ 없음
컴파일 에러: 🔴 N건 / ✅ 없음
폴더 구조: ✅ 정상 / 🟡 미준수
.gitignore: ✅ 정상 / 🟡 Library/ 미제외

### 종합 판단
🔴 필수 수정 N건 / 🟡 권장 수정 N건
```

---

### 3단계: 하네스 게이트

코드 품질 + 빌드 체크 결과를 합산해서 판정:

| 결과 | 처리 |
|------|------|
| 🔴 필수 수정 0건 | "verifier 에이전트로 최종 검증 후 passes:true 처리를 권장합니다" 안내 |
| 🔴 필수 수정 1건 이상 | `passes: false` 유지 + 우선 수정 항목 목록 출력 |

> passes:true 변경은 verifier 에이전트만 수행한다. /audit은 품질 게이트 역할만 한다.
> 🟡 권장 수정은 passes 판정에 영향을 주지 않는다.

---

## migrate 모드

`/audit migrate` 실행 시 이 섹션을 수행한다.

### 목적
Unity 구형 API 사용 위치를 탐지하고 교체 방법을 안내한다. 수정은 사용자 확인 후 진행.

### 탐지 대상

| 구형 API | 신형 대체 | 심각도 |
|---------|---------|--------|
| `Input.GetKey/Button/Axis` | New Input System | 🔴 |
| `Resources.Load` | Addressables | 🔴 |
| `StartCoroutine/IEnumerator` | UniTask | 🟡 |
| `FindObjectOfType/FindObjectsOfType` | 의존성 주입·ServiceLocator | 🟡 |
| `UnityEngine.UI.Text` | TextMeshPro | 🟡 |
| `OnGUI` | UI Toolkit / uGUI | 🟡 |

### 실행

각 패턴을 Grep으로 `Assets/_Project/Scripts/` 전체 탐색:

```bash
grep -rn "Input\.GetKey\|Input\.GetButton\|Input\.GetAxis" --include="*.cs" Assets/
grep -rn "Resources\.Load" --include="*.cs" Assets/
grep -rn "StartCoroutine\|IEnumerator" --include="*.cs" Assets/
grep -rn "FindObjectOfType\|FindObjectsOfType\b" --include="*.cs" Assets/
grep -rn "UnityEngine\.UI\.Text\b" --include="*.cs" Assets/
grep -rn "void OnGUI" --include="*.cs" Assets/
```

### 결과 출력

```
## 구형 API 탐지 결과

### 🔴 즉시 교체 권장 (N건)
| 파일 | 줄 | 구형 API | 교체 대상 |
|------|----|---------|---------|

### 🟡 교체 권장 (N건)
| 파일 | 줄 | 구형 API | 교체 대상 |
|------|----|---------|---------|

### 교체 방법 요약
[탐지된 항목별 교체 코드 예시 제시]
```

탐지 0건이면: `✅ 구형 API 없음 — 모든 패턴이 최신 기준에 부합합니다`

---

## localize 모드

`/audit localize` 실행 시 이 섹션을 수행한다.

### 목적
.cs 파일에서 하드코딩된 UI·게임 문자열을 탐지하고 LocalizationKey 적용 대상을 식별한다.

### 제외 대상 (로컬라이즈 불필요)
- `Debug.Log/Warning/Error` 내부 문자열
- 파일 경로, URL, 태그명
- `const`·`static readonly` 문자열 상수
- 주석

### 탐지 실행

```bash
grep -rn '"[가-힣a-zA-Z ]\{4,\}"' --include="*.cs" Assets/
```

탐지 결과에서 제외 패턴 필터링 후 분류:
- **UI 텍스트** (`SetText`, `text =`, `TMP_Text`) → 🔴 로컬라이즈 필수
- **게임 메시지** (팝업, 알림, 힌트) → 🔴 로컬라이즈 필수
- **기타** → 🟢 검토 필요

### LocalizationKey 네이밍 제안

```
UI/Button/Start         ← UI 버튼
UI/Label/Score          ← UI 레이블
Game/Message/LevelClear ← 게임 메시지
Game/Hint/Step01        ← 힌트
```

### 결과 출력

```
## 하드코딩 문자열 탐지 결과

### 🔴 로컬라이즈 필수 (N건)
| 파일 | 줄 | 문자열 | 제안 Key |
|------|----|--------|---------|

### 🟢 검토 필요 (N건)

### 다음 단계
Unity Localization 패키지 미설치 시: Package Manager → Localization 설치 안내
```

탐지 0건이면: `✅ 하드코딩 문자열 없음 — 로컬라이즈 준비 완료 상태입니다`
