# Unity 컴포넌트 캐싱 & 이벤트 패턴

## 컴포넌트 캐싱 (필수)

Awake에서 캐싱 필수. TryGetComponent 권장 (GC 없음):

```csharp
private Rigidbody m_rb;
private Renderer m_childRenderer;

private void Awake()
{
    m_rb = GetComponent<Rigidbody>();
    m_childRenderer = GetComponentInChildren<Renderer>();
}

// ✅ TryGetComponent — Editor에서 컴포넌트 미존재 시 allocation 없음
if (TryGetComponent<Rigidbody>(out var rb)) { rb.AddForce(Vector3.up); }
```

---

## 이벤트 구독/해제 패턴

```csharp
// System.Action 권장 (성능 우위)
public event Action<float> OnHealthChanged;

private void OnEnable()  { GameEvents.OnStageCleared += HandleStageCleared; }
private void OnDisable() { GameEvents.OnStageCleared -= HandleStageCleared; } // 반드시 해제
```

> `OnDisable`에서 구독 해제 누락 시 메모리 릭 유발. Critical 이슈.
