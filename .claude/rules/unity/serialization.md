# Unity 직렬화 규칙

## [RequireComponent] — 컴포넌트 의존성 명시

```csharp
[RequireComponent(typeof(Rigidbody))]
[RequireComponent(typeof(Animator))]
public class PlayerController : MonoBehaviour { }
```

---

## 직렬화 규칙

```csharp
// [SerializeField] private 필수, public 필드 직접 노출 금지
[Range(0f, 100f)]
[Tooltip("체력 (0~100)")]
[SerializeField] private float m_health = 100f;

// 관련 데이터 그룹화 — Serializable 중첩 클래스
// [Serializable] 데이터 클래스는 public 필드 허용 (Inspector 직렬화 목적 — m_ 접두사 예외)
[Serializable]
public class AttackData
{
    public float Damage;
    public float Cooldown;
    public AudioClip HitSound;
}
[SerializeField] private AttackData m_attackData;
```

---

## 자주 누락되는 네임스페이스

| using | 대표 타입 |
|-------|----------|
| `System` | Action, Func, IDisposable |
| `System.Collections` | IEnumerator |
| `System.Collections.Generic` | List, Dictionary |
| `UnityEngine` | MonoBehaviour, Vector3 |
| `UnityEngine.UI` | Button, Image |
| `TMPro` | TMP_Text, TextMeshProUGUI |
| `UnityEngine.Pool` | ObjectPool |
| `UnityEngine.SceneManagement` | SceneManager |
