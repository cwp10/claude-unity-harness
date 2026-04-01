---
name: unity-setup-mcp
description: >
  Unity MCP 서버(CoplayDev/unity-mcp)를 현재 프로젝트에 연동합니다.
  패키지 설치 안내, 서버 시작, MCP 등록까지 단계별로 안내합니다.
  Usage: /unity-setup-mcp
tools: Bash
keywords: [unity, mcp, setup, connect, server]
---

# /unity-setup-mcp

Unity MCP 서버(CoplayDev/unity-mcp)를 현재 프로젝트에 연동한다.

## 사전 조건 확인

먼저 아래 두 가지를 확인한다:

1. **Unity Editor 실행 중인지** — 닫혀 있으면 먼저 열도록 안내
2. **CoplayDev/unity-mcp 패키지 설치 여부** — 미설치 시 설치 안내 출력

## 단계별 실행

### 1단계 — 패키지 설치 안내 (미설치 시)

Unity Package Manager에서 Git URL로 설치:
```
https://github.com/CoplayDev/unity-mcp.git
```
Window > Package Manager > + > Add package from git URL

### 2단계 — Unity에서 서버 시작

Unity 메뉴: **Window > MCP For Unity** 창 열기
→ Client: **Claude Code** 선택
→ **Start Server** 클릭
→ 🟢 Session 연결 확인

### 3단계 — MCP 서버 등록 (자동)

아래 명령을 실행하여 현재 프로젝트에 UnityMCP를 등록한다:

```bash
claude mcp add --scope local --transport http UnityMCP http://127.0.0.1:8080/mcp
```

이미 등록된 경우 아래 메시지가 출력된다:
```
MCP server UnityMCP already exists
```
→ 이 경우 4단계로 넘어간다.

### 4단계 — 연결 확인

포트 응답 확인:
```bash
# Windows PowerShell
Test-NetConnection -ComputerName 127.0.0.1 -Port 8080

# 결과: TcpTestSucceeded : True 이어야 함
```

`/mcp` 명령으로 UnityMCP · ✓ connected 확인

### 5단계 — 완료 보고

연결 성공 시:
```
✅ UnityMCP 연결 완료
이제 Claude Code에서 Unity Editor를 직접 제어할 수 있습니다.

사용 예시:
- "현재 씬 계층 구조 보여줘"
- "Player 태그 오브젝트 찾아줘"
- "EditMode 테스트 실행해줘"
- "빈 GameObject 만들어줘"
```

## 주의사항

- 등록은 **프로젝트 로컬 스코프** (`--scope local`) — 다른 Unity 프로젝트에서는 해당 프로젝트 내에서 `/unity-setup-mcp` 재실행 필요
- Unity Editor가 꺼지면 MCP 연결이 끊김 → Editor 재시작 후 Start Server 다시 클릭
- 포트 충돌 시 Unity MCP For Unity 창에서 포트 변경 가능 (기본: 8080)

## MCP 미연결 상태에서의 동작

UnityMCP 툴 호출 시:
```
Error: Failed to connect to MCP server UnityMCP
Connection refused: http://127.0.0.1:8080/mcp
```
→ Unity Editor 실행 + Start Server 클릭으로 해결
