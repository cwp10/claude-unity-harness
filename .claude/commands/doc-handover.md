---
description: >
  특정 시스템의 인수인계 문서를 자동 생성한다.
  사용법: /doc-handover [시스템명 또는 폴더경로]
  예시:  /doc-handover CombatSystem
  예시:  /doc-handover Assets/_Project/Scripts/Game/Combat/
  예시:  /doc-handover src/app/dashboard/
allowed-tools: Read, Glob, Grep, Write
---

대상: $ARGUMENTS

## 실행 순서

### 1. 프로젝트 타입 확인
`.claude/CLAUDE.md` 읽기 → @import 줄로 Unity/웹 판단

### 2. 관련 파일 수집

**Unity 프로젝트:**
- 인자가 폴더면: 해당 폴더의 `.cs` 파일 전체 수집 (Glob)
- 인자가 클래스명이면: Grep으로 관련 `.cs` 파일 탐색
- 관련 ScriptableObject 경로 탐색 (Grep으로 SO_ 패턴 검색)

**웹 프로젝트:**
- 인자가 폴더면: 해당 폴더의 `.ts` `.tsx` `.js` 파일 수집 (Glob)
- 인자가 컴포넌트/모듈명이면: Grep으로 관련 파일 탐색
- 관련 타입 정의·API 라우트 파악

### 3. 코드 분석
각 파일을 Read로 읽어 다음을 파악:
- 역할 (주석 + 클래스명·함수명으로 파악)
- public API 목록
- 외부 의존성
- TODO / FIXME 주석 수집

### 4. 문서 생성
`docs/guide/HANDOVER_[시스템명]_[YYYYMMDD].md`로 저장.

```markdown
# [시스템명] 인수인계 문서

작성일: YYYY-MM-DD
대상 독자: 해당 스택 개발 경험 있는 외주 개발자

---

## 1. 시스템 개요
(코드에서 파악한 역할을 2~3문장으로)

## 2. 관련 파일 목록
| 파일 경로 | 역할 | 수정 빈도 예상 |
|----------|------|--------------|
| (수집된 파일 전체) | | |

## 3. 설정값 위치
Unity: 관련 ScriptableObject 목록과 역할
웹:   관련 환경변수·config 파일

## 4. 주요 작업 How-To
(가장 자주 할 작업을 단계별로 추론해서 작성)

## 5. 알려진 이슈 및 주의사항
(TODO / FIXME 목록 + 코드에서 발견한 주의사항)

## 6. 확장 포인트
Unity: interface, abstract class, virtual 메서드 목록
웹:   props interface, 훅 파라미터, 확장 가능한 컴포넌트

## 7. 의존성 맵
(이 시스템이 참조하는 것, 이 시스템을 참조하는 것)

## 8. 연락처
(작성자 칸)
```

### 5. 결과 보고
```
## 인수인계 문서 생성 완료

저장 위치: docs/guide/HANDOVER_[시스템명]_[날짜].md
분석한 파일: N개
발견된 TODO/FIXME: N건

채워 넣어야 할 항목:
- [ ] 연락처
- [ ] 예상 수정 빈도 (테이블)
- [ ] 추가 주의사항
```
