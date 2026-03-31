# SOLID 원칙 코드 레퍼런스 (Unity)

## S — 단일 책임 원칙 (SRP)

```csharp
// ❌ 하나가 다 하는 클래스
public class UnrefactoredPlayer : MonoBehaviour
{
    // 입력 + 이동 + 오디오 + 체력 모두 혼재
}

// ✅ 역할별 분리
public class Player : MonoBehaviour { }         // 조율만
public class PlayerInput : MonoBehaviour { }    // 입력
public class PlayerMovement : MonoBehaviour { } // 이동
public class PlayerAudio : MonoBehaviour { }    // 오디오
```
목표: 클래스 200~300줄 이하 유지

---

## O — 개방-폐쇄 원칙 (OCP)

```csharp
// 추상 기본 클래스 — 기존 코드 수정 없이 새 타입 추가
public abstract class Shape
{
    public abstract float CalculateArea();
}

public class Rectangle : Shape
{
    public override float CalculateArea() => width * height;
}

public class Circle : Shape
{
    public override float CalculateArea() => radius * radius * Mathf.PI;
}

// 계산기는 변경 없이 모든 Shape 처리
public class AreaCalculator
{
    public float GetArea(Shape shape) => shape.CalculateArea();
}
```

---

## L — 리스코프 치환 원칙 (LSP)

```csharp
// 차량과 기차는 TurnLeft/TurnRight를 공유할 수 없으므로 분리
public interface IMovable  { void GoForward(); }
public interface ITurnable { void TurnLeft(); void TurnRight(); }

public class RoadVehicle : IMovable, ITurnable { ... }  // 차는 방향 전환 가능
public class RailVehicle  : IMovable { ... }            // 기차는 방향 전환 불가

public class Car   : RoadVehicle { ... }
public class Train : RailVehicle { ... }
```
핵심: 서브 클래스가 기본 클래스 메서드를 비워두면 LSP 위반

---

## I — 인터페이스 분리 원칙 (ISP)

```csharp
// 작고 집중된 인터페이스로 분리
public interface IMovable    { float MoveSpeed { get; } void GoForward(); }
public interface IDamageable { float Health { get; }    void Die(); }
public interface IExplodable { void Explode(); }

// 필요한 인터페이스만 구현
public class ExplodingBarrel : MonoBehaviour, IDamageable, IExplodable { }
public class EnemyUnit       : MonoBehaviour, IDamageable, IMovable    { }
```
주의: Unity는 인터페이스를 직접 직렬화 불가 → 구상 클래스 직렬화 후 런타임에 is 키워드 캐스팅

---

## D — 종속성 역전 원칙 (DIP)

```csharp
// ❌ Switch가 Door에 직접 의존
public class Switch { public Door m_door; }

// ✅ ISwitchable 인터페이스로 추상화
public interface ISwitchable
{
    bool IsActive { get; }
    void Activate();
    void Deactivate();
}

public class Switch
{
    public ISwitchable m_client;  // Door뿐 아니라 ISwitchable 구현체 모두 가능

    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.E))
        {
            if (m_client.IsActive) m_client.Deactivate();
            else m_client.Activate();
        }
    }
}
```

---

## 추상 클래스 vs. 인터페이스 선택 기준

| 상황 | 선택 |
|------|------|
| 핵심 공유 기능 + 기본 구현 필요 | 추상 클래스 |
| 유연한 다중 구현 필요 | 인터페이스 |
| Unity 직렬화 필요 | 추상 클래스 (인터페이스 직렬화 불가) |
| 여러 계층에 걸친 공통 계약 | 인터페이스 |
