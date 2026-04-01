---
name: unity-scene-audit
description: >
  현재 Unity 씬의 구조를 분석하여 품질 이슈를 보고합니다.
  누락 컴포넌트, 네이밍 규칙 위반, 성능 이슈, 구조적 문제를 점검합니다.
  UnityMCP manage_scene, find_gameobjects 툴을 사용합니다.
  Usage: /unity-scene-audit
tools: mcp__UnityMCP__manage_scene, mcp__UnityMCP__find_gameobjects, mcp__UnityMCP__manage_components, mcp__UnityMCP__read_console, Read, Glob
triggers:
  - "씬 검사"
  - "씬 점검"
  - "씬 감사"
  - "scene audit"
keywords: [scene, audit, quality, hierarchy, gameobject, missing]
---

# /unity-scene-audit

현재 Unity 씬 전체를 분석하여 품질 이슈를 보고한다.

## MCP 연결 확인

UnityMCP `manage_scene` 툴을 호출한다.

**미연결 시:**
```
[unity-scene-audit] UnityMCP가 연결되지 않았습니다.

해결 방법:
1. Unity Editor 실행
2. Window > MCP For Unity 열기
3. Start Server 클릭
4. /unity-scene-audit 재실행
```
→ 여기서 종료.

## 실행 순서

### 1단계: 씬 정보 수집

- `manage_scene` — 현재 씬 이름·경로·계층 구조 조회
- `find_gameobjects` — 전체 오브젝트 목록 수집
- `read_console` — 씬 관련 에러·경고 수집

### 2단계: 항목별 점검

아래 항목을 순서대로 점검한다.

#### A. 필수 오브젝트 확인
- Main Camera 존재 여부 (Tag: MainCamera)
- Directional Light 존재 여부
- EventSystem 존재 여부 (UI 씬인 경우)

#### B. Missing 탐지
- Missing Script 컴포넌트가 있는 오브젝트
- Missing Prefab Reference (Prefab 연결 끊긴 오브젝트)

#### C. 네이밍 규칙 점검
프리팹 인스턴스 네이밍 확인:
- `PFB_` 접두사 없이 사용된 프리팹
- `GameObject`, `Cube`, `Sphere` 등 기본 이름 그대로 사용된 오브젝트
- 숫자 접미사 남용 (`Player (1)`, `Enemy (2)`)

#### D. 구조적 이슈
- 루트에 너무 많은 오브젝트 (10개 초과 시 그룹화 권장)
- 빈 GameObject가 의미 없이 중첩된 경우
- 비활성화된 오브젝트가 다수인 경우 (5개 초과)

#### E. 성능 체크
- 정적(Static) 설정이 누락된 배경 오브젝트
- Collider는 있지만 Rigidbody 없는 오브젝트 (의도적인지 확인)
- 씬 내 오브젝트 총 수 (500개 초과 시 경고)

### 3단계: 결과 보고

심각도 기준:
- 🔴 Critical: Missing Script·Missing Reference·카메라 없음
- 🟡 Warning: 네이밍 위반·구조적 이슈·성능 우려
- 🟢 Suggestion: 개선 가능한 부분

```
## Unity 씬 감사 결과 — [씬명]
분석 오브젝트: N개

### 🔴 Critical (N건)
- [오브젝트명]: Missing Script 발견
- ...

### 🟡 Warning (N건)
- [오브젝트명]: 기본 이름 사용 (GameObject) → 의미있는 이름으로 변경 권장
- ...

### 🟢 Suggestion (N건)
- 루트 오브젝트 N개 → [Environment], [UI], [Characters] 등으로 그룹화 권장
- ...

---
Critical N건 · Warning N건 · Suggestion N건
```

### 4단계: 수정 제안

Critical이 있으면:
```
## 즉시 수정이 필요한 항목

1. [오브젝트명] Missing Script 제거
   → Inspector에서 해당 컴포넌트 우클릭 > Remove Component
2. ...

수정 후 /unity-scene-audit 재실행하세요.
```

Critical 0건이면:
```
✅ Critical 이슈 없음. 씬 구조가 양호합니다.
Warning 항목은 /review 또는 다음 작업 시 반영하세요.
```
