# Unity 고급 패턴 레퍼런스

## 커맨드 패턴 (Command)

```csharp
public interface ICommand
{
    void Execute();
    void Undo();
}

public static class CommandInvoker
{
    private static readonly Stack<ICommand> s_undoStack = new();
    private static readonly Stack<ICommand> s_redoStack = new();

    public static void Execute(ICommand command)
    {
        command.Execute();
        s_undoStack.Push(command);
        s_redoStack.Clear();
    }

    public static void Undo()
    {
        if (s_undoStack.Count == 0) return;
        ICommand command = s_undoStack.Pop();
        command.Undo();
        s_redoStack.Push(command);
    }

    public static void Redo()
    {
        if (s_redoStack.Count == 0) return;
        ICommand command = s_redoStack.Pop();
        command.Execute();
        s_undoStack.Push(command);
    }
}
```
언제: 실행 취소/다시 실행, 입력 리플레이, 행동 기록

---

## 전략 패턴 (Strategy)

```csharp
// ScriptableObject 기반 전략 (권장)
public abstract class AttackStrategy : ScriptableObject
{
    public abstract void Execute(GameObject user, GameObject target);
}

[CreateAssetMenu(menuName = "Strategies/MeleeAttack")]
public class MeleeAttack : AttackStrategy
{
    [SerializeField] private float m_damage = 10f;
    public override void Execute(GameObject user, GameObject target)
    {
        target.GetComponent<IDamageable>()?.TakeDamage(m_damage);
    }
}

// 런타임에 전략 교체 가능
public class Character : MonoBehaviour
{
    [SerializeField] private AttackStrategy m_attackStrategy;

    public void Attack(GameObject target) => m_attackStrategy.Execute(gameObject, target);

    public void SwitchStrategy(AttackStrategy newStrategy) => m_attackStrategy = newStrategy;
}
```
언제: 런타임에 알고리즘·동작 교체 / AI 행동 패턴, 난이도 조정

---

## 플라이웨이트 패턴 (Flyweight)

```csharp
// 공유 데이터 (ScriptableObject) — 메모리 절약
[CreateAssetMenu(menuName = "Data/ShipData")]
public class ShipData : ScriptableObject
{
    public string UnitName;
    public float Speed;
    public int AttackPower;
    public Mesh Mesh;
    public Material Material;
}

// 각 인스턴스는 공유 데이터 참조 + 고유 상태만 개별 저장
public class Ship : MonoBehaviour
{
    [SerializeField] private ShipData m_sharedData;  // 공유
    [SerializeField] private float m_health;          // 고유
    [SerializeField] private Vector3 m_position;      // 고유
}
```
언제: 동일 데이터를 가진 오브젝트 대량 생성 시 메모리 최적화

---

## 더티 플래그 (Dirty Flag)

```csharp
public class Sector : MonoBehaviour
{
    public bool IsDirty { get; private set; }

    public void MarkDirty() => IsDirty = true;
    public void Clean()     => IsDirty = false;
}

// 매 프레임 변화 감지 → 더티 표시 → 더티할 때만 비싼 연산
public class SectorManager : MonoBehaviour
{
    private void Update()
    {
        bool isPlayerClose = CheckPlayerProximity();
        if (isPlayerClose != m_sector.IsLoaded)
            m_sector.MarkDirty();

        if (m_sector.IsDirty)
        {
            UpdateSector();   // 비용 높은 연산
            m_sector.Clean();
        }
    }
}
```
언제: 씬 로딩/언로딩, AI 경로 재계산, UI 레이아웃 갱신 등 비용 높은 연산 최소화

---

## SO_Config (밸런스 데이터)

```csharp
[CreateAssetMenu(menuName = "Config/Enemy")]
public class SO_EnemyConfig : ScriptableObject
{
    public float MaxHealth   = 100f;
    public float MoveSpeed   = 3f;
    public float AttackDamage = 10f;
    public float AttackRange = 2f;
}
```
언제: 기획자가 수정할 밸런스 수치 / 피할 때: 런타임에 자주 바뀌는 값

---

## MVP (UI 패턴)

```csharp
// View: 표시만, 로직 없음
public class InventoryView : MonoBehaviour
{
    public event Action<int> OnSlotClicked;
    public void Refresh(IReadOnlyList<ItemData> items) { ... }
}

// Presenter: Model↔View 중개
public class InventoryPresenter
{
    private readonly InventorySystem m_model;
    private readonly InventoryView m_view;

    public InventoryPresenter(InventorySystem model, InventoryView view)
    {
        m_model = model;
        m_view  = view;
        m_view.OnSlotClicked += OnSlotClicked;
        m_model.OnInventoryChanged += RefreshView;
    }

    private void RefreshView()          => m_view.Refresh(m_model.Items);
    private void OnSlotClicked(int idx) => m_model.SelectItem(idx);
}
```

---

## SOLID 원칙 코드 예시

상세 코드 예시는 `references/solid-principles.md` 참조.
필요 시 Read 도구로 해당 파일을 로드한다.

| 원칙 | 핵심 |
|------|------|
| SRP | 클래스 하나 = 역할 하나. 200~300줄 이하 유지 |
| OCP | 추상 클래스로 확장, 기존 코드 수정 불필요 |
| LSP | 서브 클래스는 기본 클래스로 완전 대체 가능해야 함 |
| ISP | 작고 집중된 인터페이스로 분리 |
| DIP | 인터페이스로 추상화, 구상 클래스에 직접 의존 금지 |

