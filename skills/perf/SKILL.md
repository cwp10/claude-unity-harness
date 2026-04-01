---
name: perf
description: >
  Unity C# 성능 패턴을 탐지해서 GC Alloc·Draw Call·Update 루프 위험 요소를 보고합니다.
  /audit보다 성능에 특화된 심층 분석입니다.
  Usage: /perf [파일 또는 폴더] (기본: 최근 변경 파일)
  Example: /perf
  Example: /perf Assets/_Project/Scripts/Player/
  Example: /perf Assets/_Project/Scripts/Game/EnemyController.cs
tools: Read, Glob, Grep, Bash
model: sonnet
triggers:
  - "성능"
  - "GC"
  - "프레임 드랍"
  - "최적화"
  - "느려"
  - "버벅"
keywords: [perf, performance, gc, alloc, drawcall, optimization, mobile]
---

대상: $ARGUMENTS

## 실행 순서

### 1단계: 분석 대상 결정

- `$ARGUMENTS` 있음 → 지정 파일·폴더 대상
- 없음 → `git diff --name-only HEAD`로 최근 변경 .cs 파일 대상
- 변경 파일도 없으면 → `Assets/_Project/Scripts/` 전체 .cs

### 2단계: 성능 패턴 탐지 (Grep 병렬 실행)

아래 패턴을 동시에(concurrently) 탐색한다:

**A. Update 루프 GC Alloc**
```bash
# new 키워드 (List, Dictionary, Vector 등 제외 — 필드 초기화는 허용)
grep -n "void Update\|void LateUpdate\|void FixedUpdate" --include="*.cs" -A 50 -r [대상]
```
Update 메서드 내부에서:
- `new` 키워드 사용 (GC Alloc 유발)
- LINQ (`Where`, `Select`, `OrderBy` 등)
- `string` 연결 (`+` 연산자, `string.Format`)
- `foreach` on non-array collections (boxing 위험)

**B. 컴포넌트 캐싱 누락**
```bash
grep -n "GetComponent\|GetComponentInChildren\|GetComponentInParent" --include="*.cs" -r [대상]
```
- `Update/LateUpdate/FixedUpdate` 내부 호출 → 🔴 Critical
- `Start/Awake`에서 캐싱 없이 반복 호출 → 🟡 Warning

**C. 오브젝트 탐색 남용**
```bash
grep -n "FindObjectOfType\|FindObjectsOfType\|GameObject.Find\|GameObject.FindWithTag" --include="*.cs" -r [대상]
```
- `Update` 내부 → 🔴 Critical
- `Start/Awake` 외부 → 🟡 Warning

**D. 이벤트 누수 위험**
```bash
grep -n "+= \|OnEnable\|OnDisable\|OnDestroy" --include="*.cs" -r [대상]
```
- `+=` 구독 후 `-=` 해제 누락 → 🟡 Warning

**E. Draw Call 위험**
```bash
grep -n "Instantiate\|Destroy\b" --include="*.cs" -r [대상]
```
- `Update` 내 `Instantiate/Destroy` → 🔴 Critical (ObjectPool 미사용)

> `Resources.Load` · `FindObjectOfType` API 교체 여부는 `/audit migrate`에서 담당한다.

### 3단계: 결과 출력

```
## 성능 분석 결과

### 🔴 Critical (N건)
| 파일 | 줄 | 패턴 | 문제 |
|------|----|------|------|
| PlayerController.cs | 42 | GetComponent in Update | 매 프레임 컴포넌트 탐색 |

### 🟡 Warning (N건)
| 파일 | 줄 | 패턴 | 문제 |
|------|----|------|------|

### 🟢 Suggestion (N건)

### 종합
- 분석 파일: N개
- GC Alloc 위험: N건
- Draw Call 위험: N건
- 캐싱 누락: N건
```

Critical 0건이면:
```
✅ 성능 Critical 없음 — 분석된 N개 파일에서 즉각 수정이 필요한 항목 없음
```

### 4단계: 수정 안내

Critical 항목이 있으면:
- 각 항목마다 **수정 방법 코드 예시** 제시
- 일괄 수정 원하면 `/fix [파일경로]` 사용 안내
