# Unity 프로젝트 설정 (claude-unity 플러그인)

## 정체성
Unity 개발자.

## 규칙
@rules/response.md
@rules/git.md
@rules/code-review.md

## 스택 규칙 (setup 실행 후 활성화)
# 아래 3줄은 /setup 실행 시 자동으로 활성화됩니다:
# @engines/unity.md
# @languages/csharp.md
# @domains/unity.md

## 프로젝트 초기화
/setup 실행 시 이 파일 끝에 스택·버전·폴더 구조가 자동으로 추가됩니다.

## 점진적 진행 원칙
한 번에 1개 기능만 작업한다:
1. feature_list.json 에서 passes: false 인 항목 중 가장 높은 우선순위 1개 선택
2. 해당 기능 구현 완료 및 검증
3. passes: true 로 변경 후 /context-save 실행
4. 다음 기능으로 이동

여러 기능을 동시에 시작하지 않는다. 컨텍스트 중간에 작업이 끊기면 다음 세션에서 재개한다.

## 세션 시작 루틴
새 세션 시작 시 반드시 `/context-load` 를 실행한다.

## Compaction 지시문
/compact 실행 시 반드시 보존할 항목:
- 작업 중인 기능명과 현재 진행 단계
- 변경된 파일 목록 (경로 전체)
- 내린 설계 결정과 이유
- 발견된 Critical 이슈 목록
- 다음에 할 작업
