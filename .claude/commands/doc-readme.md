---
description: >
  현재 프로젝트의 README.md를 자동 생성한다.
  Unity 게임·산업교육·Next.js 웹 프로젝트 모두 지원.
  사용법: /doc-readme
  프로젝트 루트에서 실행 필요.
allowed-tools: Read, Glob, Grep, Bash, Write
---

## 실행 순서

### 0. 프로젝트 타입 확인
`.claude/CLAUDE.md` 를 Read해서 @import 확인:
- `engines/unity` → Unity 프로젝트 → 1단계 A 실행
- `domains/web` → 웹 프로젝트 → 1단계 B 실행
- 없으면 → 폴더 구조로 판단

### 1단계 A. Unity 프로젝트 정보 수집

**Unity 버전**
- `ProjectSettings/ProjectVersion.txt` 읽기

**패키지 목록**
- `Packages/packages-lock.json` 읽기 → 주요 패키지 추출

**빌드 타겟**
- `ProjectSettings/ProjectSettings.asset` 읽기 → 플랫폼 확인

**폴더 구조**
- `Assets/` 하위 폴더 구조 파악 (Glob, 2단계 깊이)

**씬 목록**
- `Assets/**/*.unity` 파일 목록 수집

**기존 CLAUDE.md 확인**
- `.claude/CLAUDE.md` 읽기 → 프로젝트 정보 참고

### 1단계 B. 웹 프로젝트 정보 수집

**패키지 정보**
- `package.json` 읽기 → 프레임워크·주요 패키지 추출

**폴더 구조**
- `src/` 또는 `app/` 하위 구조 파악 (Glob, 2단계 깊이)

**환경 설정**
- `.env.example` 또는 `next.config.*` 읽기 → 설정 파악

**기존 CLAUDE.md 확인**
- `.claude/CLAUDE.md` 읽기 → 프로젝트 정보 참고

### 2. 핵심 모듈 파악

**Unity 프로젝트:**
- `Assets/_Project/Scripts/Core/` 의 주요 클래스 Read
- 각 클래스의 역할을 summary 주석 또는 클래스명으로 파악

**웹 프로젝트:**
- `src/` 또는 `app/` 하위 주요 컴포넌트·훅·서비스 Read
- 각 모듈의 역할을 주석 또는 파일명으로 파악

### 3. README.md 생성
프로젝트 루트에 `README.md`로 저장.

```markdown
# [프로젝트명]

> [한 줄 설명 - ProjectSettings에서 추출]

---

## 개발 환경 (Requirements)

| 항목 | 버전 |
|------|------|
| Unity | [버전] |
| 타겟 플랫폼 | [플랫폼] |
| Render Pipeline | [URP/HDRP/Built-in] |

### 필수 패키지
| 패키지 | 버전 | 용도 |
|--------|------|------|
| (packages-lock에서 추출) | | |

---

## 빠른 시작 (Quick Start)

```bash
# 1. 레포 클론
git clone [GitHub URL]

# 2. Unity Hub에서 프로젝트 열기
# Unity [버전] 필요

# 3. 필수 초기 설정
```
[Addressables 사용 시 빌드 방법 등 자동 감지해서 추가]

---

## 프로젝트 구조

```
Assets/
[수집한 폴더 구조 + 각 폴더 역할]
```

---

## 씬 구성

| 씬 이름 | 역할 |
|---------|------|
| [씬 목록] | [추론한 역할] |

---

## 핵심 시스템

| 시스템 | 위치 | 역할 |
|--------|------|------|
| [Core 클래스 목록] | | |

---

## Git 규칙

[workflows/git.md 내용 요약]

---

## 주의사항

- .meta 파일 반드시 커밋
- Library/, Temp/, Builds/ 는 .gitignore 처리
- [Addressables 사용 시 추가 주의사항]

---

## 빌드 방법

### [플랫폼명]
[빌드 절차]
```

### 4. 결과 보고
```
## README.md 생성 완료

저장 위치: ./README.md

직접 채워 넣어야 할 항목:
- [ ] GitHub URL
- [ ] 프로젝트 한 줄 설명 (자동 추출이 부정확할 수 있음)
- [ ] 씬별 역할 확인
- [ ] 빌드 명령어 확인
```
