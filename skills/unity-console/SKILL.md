---
name: unity-console
description: >
  Unity 콘솔 로그를 읽어 에러·경고를 분류하고 원인 분석 및 수정 방법을 제안합니다.
  UnityMCP read_console 툴을 사용합니다.
  Usage: /unity-console
tools: mcp__UnityMCP__read_console, mcp__UnityMCP__find_in_file, Read, Glob, Grep
triggers:
  - "콘솔 확인"
  - "에러 확인"
  - "유니티 에러"
  - "컴파일 에러"
keywords: [console, error, warning, unity, log, compile]
---

# /unity-console

Unity 콘솔 로그를 읽고 에러를 분석한다.

## MCP 연결 확인

UnityMCP `read_console` 툴을 호출한다.

**미연결 시:**
```
[unity-console] UnityMCP가 연결되지 않았습니다.

해결 방법:
1. Unity Editor 실행
2. Window > MCP For Unity 열기
3. Start Server 클릭
4. /unity-console 재실행
```
→ 여기서 종료.

## 실행 순서

### 1단계: 콘솔 로그 수집

`read_console` 툴 호출 — 전체 로그 수집.

### 2단계: 심각도별 분류 및 출력

수집한 로그를 아래 형식으로 출력한다:

```
## Unity 콘솔 분석

### 🔴 에러 (N건)
에러가 있으면 각 항목:
- **[CS0123 | NullReferenceException 등]** 에러 메시지
  파일: Scripts/PlayerController.cs:42
  원인: (간결한 원인 설명)
  수정: (구체적인 수정 방법 또는 코드 스니펫)

에러 없으면: `없음 ✅`

### 🟡 경고 (N건)
Warning 항목 목록 (최대 5건, 반복성 경고는 묶어서 표시)
중요도 낮은 경고면 생략 가능.

경고 없으면: `없음 ✅`

### ℹ️ 상태
- 컴파일: 성공 / 실패
- 마지막 로그 시각: HH:MM:SS
```

### 3단계: 수정 우선순위 제안

에러가 2건 이상이면:
```
## 수정 순서 제안
1. [가장 먼저 수정할 에러] — 이유
2. [다음 에러]
```

에러가 0건이면:
```
✅ 에러 없음. 경고만 N건 있습니다.
중요 경고가 있으면 /review 로 상세 리뷰를 받으세요.
```
