# Unity 컴포넌트 캐싱 & 이벤트 패턴

## 컴포넌트 캐싱 (필수)

Awake에서 캐싱 필수. TryGetComponent 권장 (GC 없음):

```csharp
private Rigidbody m_rb;
private Renderer m_childRenderer;

// [RequireComponent] 선언 시 → Awake에서 GetComponent 직접 할당 안전
private void Awake()
{
    m_rb = GetComponent<Rigidbody>();
    m_childRenderer = GetComponentInChildren<Renderer>();
}

// [RequireComponent] 없이 런타임 조건부 접근 시 → TryGetComponent 사용 (GC 없음)
if (TryGetComponent<Rigidbody>(out var rb)) { rb.AddForce(Vector3.up); }
```

---

## 이벤트 구독/해제 패턴

`OnEnable`/`OnDisable` 쌍으로 구독·해제. `OnDestroy`는 `OnDisable` 미호출 상황의 보험용.

```csharp
// System.Action 권장 (성능 우위)
public event Action<float> OnHealthChanged;

private void OnEnable()  { GameEvents.OnStageCleared += HandleStageCleared; }
private void OnDisable() { GameEvents.OnStageCleared -= HandleStageCleared; }
private void OnDestroy() { GameEvents.OnStageCleared -= HandleStageCleared; } // 보험용
```
