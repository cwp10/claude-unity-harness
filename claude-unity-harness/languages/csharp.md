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
| public 필드·프로퍼티 | PascalCase | `MaxHealth`, `DamageMultiplier` |
| private 필드 | camelCase | `currentHealth` |
| 상수 | PascalCase | `MaxItems` |
| 파라미터·로컬 변수 | camelCase | `damageAmount` |
| bool 변수 | 동사 접두사 | `isDead`, `isWalking` |
| 이벤트 (발생 전) | 현재진행형 | `OpeningDoor` |
| 이벤트 (발생 후) | 과거분사 | `DoorOpened` |
| 이벤트 발생 메서드 | On 접두사 | `OnDoorOpened()` |

> Unity 프로젝트의 필드 접두사 (m\_, k\_, s\_) 규칙은 engines/unity.md 참조

```csharp
// C# 기본 네이밍
public float DamageMultiplier = 1.5f;
private bool isDead;
private float currentHealth;
public const int MaxItems = 100;

// 나쁜 예
int d;            // 무의미한 이름
bool dead;        // 동사 접두사 누락
float HP;         // 약어 지양
```

---

## 열거형

```csharp
// 이름과 값 모두 PascalCase, 단수 명사
public enum WeaponType { Knife, Gun, RocketLauncher }

// [Flags] 비트 열거형만 복수형
[Flags]
public enum AttackModes { None = 0, Melee = 1, Ranged = 2, Special = 4 }
```

---

## 포맷 규칙

### 중괄호 — Allman 스타일 (필수)
```csharp
// ✅ 여는 중괄호는 항상 새 줄
void DoSomething()
{
    if (condition)
    {
        Execute();
    }
}

// ❌ 단일 줄도 중괄호 생략 금지
for (int i = 0; i < 100; i++) DoSomething(i);
```

### 프로퍼티
```csharp
public int MaxHealth => m_maxHealth;                            // 읽기 전용
public int Health { get => m_health; set => m_health = value; }
public int Score { get; private set; }                          // 쓰기 제한
```

### 공백·줄 너비
- 줄 너비: 80~120자 권장
- 메서드 파라미터 쉼표 뒤 공백 하나
- 비교 연산자 앞뒤 공백 하나
- 괄호와 인수 사이 공백 없음

---

## 클래스 구성 순서 (상위 수준 먼저)

```
1. 상수 (const, static readonly)
2. [SerializeField] private 필드
3. 프로퍼티
4. 이벤트·델리게이트
5. 생명주기 메서드 (엔진별 규칙 참조)
6. public 메서드
7. private 메서드
```

---

## 메서드 규칙
- **작고 하나의 책임**만 가져야 함
- 인수는 적게 (3개 초과 시 분리 검토)
- 플래그 전달 대신 별도 메서드 생성
- bool 반환 메서드는 질문 형식: `IsGameOver()`, `HasStartedTurn()`

```csharp
// ❌ WET (중복)
private void PlayExplosionA(Vector3 pos) { ... }
private void PlayExplosionB(Vector3 pos) { ... }

// ✅ DRY
private void PlayFX(ParticleSystem particle, AudioClip clip, Vector3 pos) { ... }
```

---

## 주석 규칙
- 코드로 알 수 없는 **이유**만 주석
- public API: `/// <summary>` XML 태그 필수
- 주석 처리한 코드는 삭제 (소스 관리로 이전 코드 확인)
- TODO는 최신 유지, 오래된 것은 삭제
- 헝가리언 표기법 사용 안 함

```csharp
// ❌ 코드가 말해주는 것 반복
// 표적이 될 대상
Transform targetToShoot;

// ✅ 이유를 설명
// 성능상 캐싱 필요 — Update에서 GetComponent 호출 금지
private Rigidbody m_rb;

/// <summary>
/// 플레이어에게 데미지를 입힙니다. Apply damage to the player.
/// </summary>
public void TakeDamage(float amount) { ... }
```

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
