# Unity 스크립트 규칙

## 버전 기준
- Unity 6 LTS (6000.0.x) / C# 9.0 / .NET Standard 2.1
- Render Pipeline: 프로젝트별 오버라이드 (기본 URP)

---

## Unity 전용 네이밍 규칙

C# 기본 규칙(csharp.md) 위에 Unity 프로젝트 전용 규칙을 추가 적용한다.

### 필드 접두사 (Unity 공식 스타일 가이드 기준)

| 대상 | 접두사 | 예시 |
|------|--------|------|
| private 멤버 변수 | `m_` | `m_currentHealth`, `m_movementSpeed` |
| 상수 | `k_` | `k_MaxItems`, `k_DefaultSpeed` |
| 정적 변수 | `s_` | `s_instance`, `s_count` |

```csharp
// Unity 프로젝트 네이밍
private float m_currentHealth;
private bool m_isDead;
private static GameManager s_instance;
public const int k_MaxItems = 100;
private const float k_GravityScale = 9.8f;
```

### 이벤트 핸들러 명명
- 이벤트 발생 메서드: `On` 접두사 → `OnDoorOpened()`
- 관찰자의 처리 메서드: `주체이름_이벤트이름` → `GameEvents_DoorOpened`

### 에셋 네이밍
| 에셋 | 접두사 | 예시 |
|------|--------|------|
| 씬 | `Scene_` | `Scene_MainStage` |
| 프리팹 | `PFB_` | `PFB_Player` |
| ScriptableObject | `SO_` | `SO_EnemyConfig` |
| 애니메이션 | `ANIM_` | `ANIM_Player_Idle` |
| 머티리얼 | `MAT_` | `MAT_Player` |
| 텍스처 | `TEX_` | `TEX_Player_Diffuse` |

---

## MonoBehaviour 생명주기 순서

```
Awake → OnEnable → Start → Update → LateUpdate → FixedUpdate
OnDisable → OnDestroy
```

```csharp
// 클래스 구성 순서 (Unity 전용)
public class PlayerController : MonoBehaviour
{
    // 1. 상수
    private const int k_MaxJumpCount = 2;

    // 2. SerializeField
    [SerializeField] private float m_moveSpeed = 5f;
    [Tooltip("점프력 (N)")]
    [SerializeField] private float m_jumpForce = 10f;

    // 3. 프로퍼티
    public float MoveSpeed => m_moveSpeed;

    // 4. 이벤트
    public event Action OnPlayerDied;

    // 5. 생명주기
    private void Awake() { CacheComponents(); }
    private void OnEnable() { SubscribeEvents(); }
    private void Start() { Initialize(); }
    private void Update() { HandleInput(); }
    private void OnDisable() { UnsubscribeEvents(); }

    // 6. public 메서드
    public void TakeDamage(float amount) { ... }

    // 7. private 메서드
    private void CacheComponents() { ... }
}
```

---

## using 지시문 필수 체크 (코드 작성·수정 시 항상 확인)

**.cs 파일을 새로 만들거나 수정한 후, 아래 순서로 using 누락을 반드시 점검한다.**

1. 작성한 코드에서 사용한 **모든 타입**을 훑는다
2. 각 타입이 어느 네임스페이스 소속인지 확인한다
3. 파일 상단 `using` 목록과 대조한다
4. 누락된 네임스페이스가 있으면 파일 상단에 즉시 추가한다

> 타입의 네임스페이스를 확실히 모를 경우, 추측하지 말고 해당 타입이 정의된 파일을 Grep으로 찾아 확인한다.
> ```bash
> grep -rn "class 타입명\|struct 타입명\|enum 타입명\|interface 타입명" Assets/ --include="*.cs"
> ```

---

## 절대 금지

- `Update()`에서 `GetComponent()` 호출 → Awake에서 캐싱
- `FindObjectOfType()` / `FindObjectsOfType()` 남용
- `Resources.Load()` → Addressables 사용
- `Update()`에서 `new` 키워드 → GC Alloc 유발
- `public` 필드 직접 노출 → `[SerializeField] private` 사용
- 구형 `Input.GetKey()` → New Input System 사용
- `OnDisable/OnDestroy`에서 이벤트 구독 해제 누락

---

## 직렬화 규칙

```csharp
// Inspector 노출: SerializeField + private
[SerializeField] private float m_speed = 5f;

// 범위 제한
[Range(0f, 100f)]
[SerializeField] private float m_health = 100f;

// Tooltip으로 설명 (주석 대신)
[Tooltip("The amount of side-to-side friction.")]
[SerializeField] private float m_grip;

// 관련 데이터 그룹화
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

## 컴포넌트 캐싱 (필수)

```csharp
private Rigidbody m_rb;
private Animator m_anim;
private AudioSource m_audio;

private void Awake()
{
    m_rb = GetComponent<Rigidbody>();
    m_anim = GetComponent<Animator>();
    m_audio = GetComponent<AudioSource>();

    // 자식·부모 컴포넌트
    m_childRenderer = GetComponentInChildren<Renderer>();
}
```

---

## 이벤트 구독/해제 패턴

```csharp
// System.Action 권장 (성능 우위)
public event Action<float> OnHealthChanged;

private void OnEnable()
{
    GameEvents.OnStageCleared += HandleStageCleared;
}

private void OnDisable()
{
    // 반드시 해제 — 메모리 누수 방지
    GameEvents.OnStageCleared -= HandleStageCleared;
}
```

---

## 비동기 처리 (UniTask)

```csharp
// Coroutine 대신 UniTask 사용
public async UniTask LoadSceneAsync(string sceneName, CancellationToken ct)
{
    await SceneManager.LoadSceneAsync(sceneName)
        .ToUniTask(cancellationToken: ct);
}

private CancellationTokenSource m_cts;
private void OnEnable()  => m_cts = new CancellationTokenSource();
private void OnDisable() { m_cts?.Cancel(); m_cts?.Dispose(); }
```

---

## SOLID 원칙 적용 (Unity 맥락)

상세 코드 예시는 unity-patterns 스킬의 `references/solid-principles.md` 참조.

| 원칙 | Unity 핵심 적용 |
|------|----------------|
| SRP | Player → PlayerInput, PlayerMovement, PlayerAudio 분리 |
| OCP | 추상 클래스로 확장, 기존 코드 수정 불필요 |
| LSP | 서브 클래스가 기본 클래스로 완전 대체 가능해야 함 |
| ISP | IMovable, IDamageable 등 작고 집중된 인터페이스 분리 |
| DIP | 인터페이스로 추상화, 구상 클래스에 직접 의존 금지 |

---

## 사용 패키지 기준
- 비동기: UniTask (Coroutine 사용 금지)
- 에셋 관리: Addressables (Resources.Load 금지)
- 트위닝: DOTween Pro
- 카메라: Cinemachine
- 입력: Input System (New)

---

## 성능 기준 (모바일)
- Draw Call: 100 이하
- GC Alloc per frame: 0 목표
- 프레임 예산: 16.6ms (60fps)
- 텍스처: ASTC 압축, Atlas 사용
- 반복 생성 객체: ObjectPool 필수

---

## 패턴 코드는 unity-patterns 스킬 참조
