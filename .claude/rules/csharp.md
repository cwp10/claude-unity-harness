---
paths:
  - "**/*.cs"
---

# C# 언어 스타일 가이드 (Unity 공식 기준)

## 개발 원칙
- **KISS**: 가장 간단한 방법으로 문제 해결
- **YAGNI**: 현재 필요한 기능만 구현
- **DRY**: 중복 로직 제거, 핵심 기능은 하나의 메서드로
- **완벽보다 더 낫게**: 기준 충족하면 커밋하고 다음으로

---

## 네이밍 컨벤션

| 대상 | 규칙 | 예시 |
|------|------|------|
| 클래스·구조체 | PascalCase | `PlayerController` |
| 인터페이스 | I + 형용사 | `IMovable`, `IDamageable` |
| 메서드 | PascalCase 동사구 | `GetDirection`, `TakeDamage` |
| public 필드·프로퍼티 | PascalCase | `MaxHealth` |
| private 필드 | camelCase | `currentHealth` |
| 상수 | PascalCase | `MaxItems` |
| 파라미터·로컬 변수 | camelCase | `damageAmount` |
| bool 변수 | 동사 접두사 | `isDead`, `isWalking` |
| 이벤트 (발생 전/후) | 현재진행형 / 과거분사 | `OpeningDoor` / `DoorOpened` |
| 이벤트 핸들러 (관찰자) | `주체이름_이벤트이름` | `GameEvents_DoorOpened` |

> Unity 필드 접두사 (`m_`, `k_`, `s_`) 규칙은 unity-engine.md 참조
> 포맷·중괄호·열거형 상세는 `.claude/rules/csharp/formatting.md` 참조

---

## 클래스 구성 순서

```
1. 상수 (const, static readonly)
2. [SerializeField] private 필드
3. 프로퍼티
4. 이벤트·델리게이트
5. 생명주기 메서드
6. public 메서드
7. private 메서드
```

---

## 메서드 규칙
- **작고 하나의 책임**만 가져야 함
- 인수는 적게 (3개 초과 시 분리 검토)
- 플래그 전달 대신 별도 메서드 생성
- bool 반환: 질문 형식 `IsGameOver()`, `HasStartedTurn()`

---

## 코드 스멜 (피해야 할 것)

| 스멜 | 설명 |
|------|------|
| God Object | 너무 많은 역할의 거대한 클래스 |
| 경직성 | 작은 변경에 여러 곳 수정 필요 |
| 취약성 | 사소한 변경으로 전체 작동 멈춤 |
| 중복 코드 | 잘라 붙여넣은 로직 (DRY 위반) |
| 과도한 주석 | 모든 변수·구문에 주석 남발 |
| 임시방편 | 근본 원인 대신 증상만 덮는 코드 |
