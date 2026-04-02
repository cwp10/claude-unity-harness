---
name: allow
description: >
  도구 권한 허용/차단 토글. ON이면 Bash·Edit·Write를 자동 승인하고 git push만 차단.
  OFF이면 추가된 권한을 제거해 기본 승인 방식으로 복원.
  Usage: /allow on
  Usage: /allow off
  Usage: /allow    ← 현재 상태 확인
tools: Read, Write
model: sonnet
triggers:
  - "권한 켜줘"
  - "권한 꺼줘"
  - "자동 승인"
  - "권한 자동"
  - "allow on"
  - "allow off"
keywords: [allow, permissions, allow, deny, auto-approve]
---

서브커맨드: $ARGUMENTS

## 관리 항목 (고정값)

**ALLOW_LIST** — 자동 승인할 도구:
- `"Bash(*)"`
- `"Edit(*)"`
- `"Write(*)"`

**DENY_LIST** — 항상 차단할 항목:
- `"Bash(git push*)"`

---

## 실행 순서

### 1단계: settings.json 읽기

`.claude/settings.json` Read.

파일이 없으면 아래 기본 구조로 생성 후 진행:
```json
{
  "permissions": {
    "allow": [],
    "deny": []
  }
}
```

### 2단계: 현재 상태 판단

`permissions.allow` 배열에 `"Bash(*)"` 포함 여부로 판단:
- 포함 → **현재 ON**
- 미포함 → **현재 OFF**

### 3단계: 서브커맨드 처리

**인수 없음 → 상태만 출력 후 종료**

**`on` → 이미 ON이면:** "이미 권한이 허용 상태입니다" 출력 후 종료

**`on` → OFF 상태이면:**
1. `permissions.allow`에 ALLOW_LIST 항목 추가 (이미 있는 항목은 건너뜀)
2. `permissions.deny`에 DENY_LIST 항목 추가 (이미 있는 항목은 건너뜀)
3. 수정된 전체 JSON을 `.claude/settings.json`에 Write

**`off` → 이미 OFF이면:** "이미 권한이 기본 상태입니다" 출력 후 종료

**`off` → ON 상태이면:**
1. `permissions.allow`에서 ALLOW_LIST 항목만 제거 (나머지 항목 유지)
2. `permissions.deny`에서 DENY_LIST 항목만 제거 (나머지 항목 유지)
3. 수정된 전체 JSON을 `.claude/settings.json`에 Write

> ⚠️ 항목 제거 시 ALLOW_LIST · DENY_LIST 목록에 있는 것만 제거한다.
> 기존에 있던 다른 allow · deny 항목은 절대 건드리지 않는다.

---

## 4단계: 결과 출력

**ON 완료:**
```
✅ 권한 허용 ON

자동 승인: Bash(*) · Edit(*) · Write(*)
항상 차단: git push

⚠️  세션을 재시작해야 적용됩니다.
```

**OFF 완료:**
```
🔒 권한 허용 OFF

제거된 자동 승인: Bash(*) · Edit(*) · Write(*)
git push 차단 해제 (deny에서 제거)

⚠️  세션을 재시작해야 적용됩니다.
```

**상태 확인 (인수 없음):**
```
현재 권한 상태: ✅ ON / 🔒 OFF

allow 목록: [현재 항목 나열]
deny 목록: [현재 항목 나열]

켜려면: /allow on
끄려면: /allow off
```
