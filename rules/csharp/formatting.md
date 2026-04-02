# C# 포맷 규칙

## 중괄호 — Allman 스타일 (필수)

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

---

## 프로퍼티

```csharp
public int MaxHealth => m_maxHealth;                            // 읽기 전용
public int Health { get => m_health; set => m_health = value; }
public int Score { get; private set; }                          // 쓰기 제한
```

---

## 공백·줄 너비
- 줄 너비: 80~120자 권장
- 메서드 파라미터 쉼표 뒤 공백 하나
- 비교 연산자 앞뒤 공백 하나
- 괄호와 인수 사이 공백 없음

---

## 열거형

```csharp
// 이름과 값 모두 PascalCase, 단수 명사
public enum WeaponType { Knife, Gun, RocketLauncher }

// [Flags] 비트 열거형만 복수형
[Flags]
public enum AttackModes { None = 0, Melee = 1, Ranged = 2, Special = 4 }
```
