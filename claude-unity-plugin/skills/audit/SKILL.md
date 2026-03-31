---
name: audit
description: >
  Runs Unity code quality audit + build checks automatically.
  Usage: /audit
  Unity: code quality audit (unity-reviewer) + .meta files, compile errors, folder structure.
  Both tasks run concurrently.
allowed-tools: Read, Glob, Grep, Bash, Write
cost: opus
triggers:
  - "전체 감사"
  - "빌드 체크"
  - "전체 리뷰"
  - "프로젝트 점검"
  - "배포 전 확인"
keywords: [audit, build, quality, meta, check, ci]
---

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
| 🔴 필수 수정 0건 | feature_list.json 해당 기능 `passes: true` 로 변경 후 완료 보고 |
| 🔴 필수 수정 1건 이상 | `passes: false` 유지 + 커밋 차단 안내 + 우선 수정 항목 목록 출력 |

> 🟡 권장 수정은 passes 판정에 영향을 주지 않는다.
