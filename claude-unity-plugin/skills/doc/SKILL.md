---
name: doc
description: >
  Unity 프로젝트 문서를 생성한다. 서브커맨드로 문서 종류를 지정한다.
  Usage:
    /doc readme                   ← 프로젝트 README.md 자동 생성
    /doc handover [시스템명]      ← 외주 개발자용 인수인계 문서
    /doc delivery [버전] [납품일] ← 클라이언트 납품 문서
  Example: /doc handover CombatSystem
  Example: /doc delivery v1.0.0 2025-06-01
allowed-tools: Read, Glob, Grep, Bash, Write
---

요청: $ARGUMENTS

## 라우팅

첫 번째 인자로 서브커맨드를 결정한다:
- `readme`   → **README 생성** 섹션으로
- `handover` → **인수인계 문서** 섹션으로
- `delivery` → **납품 문서** 섹션으로
- 없거나 불명확 → 아래 안내 출력 후 종료:

```
사용법:
  /doc readme                   프로젝트 README.md 생성
  /doc handover [시스템명]      인수인계 문서 생성
  /doc delivery [버전] [납품일] 납품 문서 생성
```

---

## README 생성

### 1단계: 프로젝트 정보 수집

- `ProjectSettings/ProjectVersion.txt` → Unity 버전
- `Packages/packages-lock.json` → 주요 패키지
- `ProjectSettings/ProjectSettings.asset` → 플랫폼
- `Assets/` 하위 폴더 구조 (Glob, 2단계 깊이)
- `Assets/**/*.unity` → 씬 목록
- `.claude/CLAUDE.md` → 프로젝트 정보 참고

### 2단계: 핵심 모듈 파악

`Assets/_Project/Scripts/Core/` 의 주요 클래스 Read.
각 클래스의 역할을 summary 주석 또는 클래스명으로 파악.

### 3단계: README.md 생성

프로젝트 루트에 `README.md`로 저장.

```markdown
# [프로젝트명]

> [한 줄 설명]

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

---

## 빠른 시작 (Quick Start)

```bash
git clone [GitHub URL]
# Unity Hub에서 프로젝트 열기
```

---

## 프로젝트 구조

```
Assets/
[폴더 구조 + 각 폴더 역할]
```

---

## 씬 구성

| 씬 이름 | 역할 |
|---------|------|

---

## 핵심 시스템

| 시스템 | 위치 | 역할 |
|--------|------|------|

---

## Git 규칙

[rules/git.md 내용 요약]

---

## 주의사항

- .meta 파일 반드시 커밋
- Library/, Temp/, Builds/ 는 .gitignore 처리

---

## 빌드 방법

### [플랫폼명]
[빌드 절차]
```

### 4단계: 결과 보고

```
## README.md 생성 완료

저장 위치: ./README.md

직접 채워 넣어야 할 항목:
- [ ] GitHub URL
- [ ] 프로젝트 한 줄 설명
- [ ] 씬별 역할 확인
- [ ] 빌드 명령어 확인
```

---

## 인수인계 문서

대상: $ARGUMENTS 에서 `handover` 다음 인자

### 1단계: 관련 파일 수집

- 인자가 폴더면: 해당 폴더의 `.cs` 파일 전체 수집 (Glob)
- 인자가 클래스명이면: Grep으로 관련 `.cs` 파일 탐색
- 관련 ScriptableObject 탐색 (SO_ 패턴 Grep)

### 2단계: 코드 분석

각 파일 Read 후 파악:
- 역할 (주석 + 클래스명·함수명)
- public API 목록
- 외부 의존성
- TODO / FIXME 수집

### 3단계: 문서 생성

`docs/guide/HANDOVER_[시스템명]_[YYYYMMDD].md`로 저장.

```markdown
# [시스템명] 인수인계 문서

작성일: YYYY-MM-DD
대상 독자: Unity C# 개발 경험 있는 외주 개발자

---

## 1. 시스템 개요
(코드에서 파악한 역할 2~3문장)

## 2. 관련 파일 목록
| 파일 경로 | 역할 | 수정 빈도 예상 |
|----------|------|--------------|

## 3. 설정값 위치
관련 ScriptableObject 목록과 역할

## 4. 주요 작업 How-To
(가장 자주 할 작업을 단계별로)

## 5. 알려진 이슈 및 주의사항
(TODO / FIXME + 코드에서 발견한 주의사항)

## 6. 확장 포인트
interface, abstract class, virtual 메서드 목록

## 7. 의존성 맵
(이 시스템이 참조하는 것 / 이 시스템을 참조하는 것)

## 8. 연락처
(작성자 칸)
```

### 4단계: 결과 보고

```
## 인수인계 문서 생성 완료

저장 위치: docs/guide/HANDOVER_[시스템명]_[날짜].md
분석한 파일: N개
발견된 TODO/FIXME: N건

직접 채워 넣어야 할 항목:
- [ ] 연락처
- [ ] 예상 수정 빈도 확인
```

---

## 납품 문서

버전: $ARGUMENTS 에서 `delivery` 다음 인자

### 1단계: 프로젝트 정보 수집

- `.claude/CLAUDE.md` → 프로젝트 기본 정보
- `ProjectSettings/ProjectSettings.asset` → 번들 ID, 버전
- `Packages/packages-lock.json` → 사용 패키지
- `Builds/` 폴더 확인
- `feature_list.json` → 구현 기능 목록
- `git log --oneline -20` → 최근 작업 내역

### 2단계: 문서 생성

`docs/guide/DELIVERY_[프로젝트명]_[버전]_[날짜].md`로 저장.

```markdown
# [프로젝트명] 납품 문서

| 항목 | 내용 |
|------|------|
| 버전 | [버전] |
| 납품일 | [날짜] |
| 개발사 | [회사명] |
| 담당자 | [연락처] |

---

## 1. 납품 내역

| 항목 | 경로/링크 | 비고 |
|------|----------|------|
| 빌드 파일 | Builds/[파일명] | |
| 소스코드 | GitHub Private Repo | 접근 권한 이전 필요 |
| 에셋 원본 | (별도 전달) | |
| 이 문서 | docs/guide/DELIVERY_... | |

---

## 2. 실행 방법

[플랫폼별 설치 방법 — 비개발자 기준]

---

## 3. 구현 기능 목록

| 기능 | 구현 여부 | 비고 |
|------|----------|------|
| (feature_list에서 추출) | 완료 / 미구현 | |

---

## 4. 알려진 제한사항 및 주의사항

---

## 5. 테스트 환경

| 항목 | 내용 |
|------|------|
| 테스트 기기 | |
| OS 버전 | |

---

## 6. 유지보수 안내

- 연락처: [연락처]
- 응답 시간: 영업일 기준 1~2일

---

## 7. 저작권 및 라이선스
```

### 3단계: 결과 보고

```
## 납품 문서 생성 완료

저장 위치: docs/guide/DELIVERY_[프로젝트명]_[버전]_[날짜].md

직접 채워 넣어야 할 항목:
- [ ] 회사명 / 담당자 연락처
- [ ] 에셋 원본 전달 링크
- [ ] 테스트 기기·계정 정보
- [ ] 유지보수 단가
```
