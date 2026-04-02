# Unity 스크립트 규칙

## 버전 기준
- Unity 6 LTS (6000.0.x) / C# 9.0 / .NET Standard 2.1
- Render Pipeline: 프로젝트별 오버라이드 (기본 URP)

---

## Unity 전용 네이밍 규칙

C# 기본 규칙(csharp.md) 위에 Unity 프로젝트 전용 규칙을 추가 적용한다.

### 필드 접두사

| 대상 | 접두사 | 예시 |
|------|--------|------|
| private 멤버 변수 | `m_` | `m_currentHealth`, `m_movementSpeed` |
| 상수 | `k_` | `k_MaxItems`, `k_DefaultSpeed` |
| 정적 변수 | `s_` | `s_instance`, `s_count` |

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
Awake → OnEnable → Start → FixedUpdate → Update → LateUpdate → OnDisable → OnDestroy
```

클래스 구성: 상수 → SerializeField → 프로퍼티 → 이벤트 → 생명주기 → public → private

---

## using 지시문 필수 체크

**.cs 작성·수정 후 반드시:**
1. 사용한 모든 타입의 네임스페이스 확인
2. 파일 상단 `using` 목록과 대조, 누락 시 즉시 추가
3. 확실치 않으면 추측 금지 — Grep으로 타입 정의 파일 확인

---

## 절대 금지

- `Update()`에서 `GetComponent()` 호출 → Awake에서 캐싱
- `FindObjectOfType()` / `FindObjectsOfType()` → `FindAnyObjectByType<T>()` / `FindObjectsByType<T>()` (Unity 6 필수)
- `Update()`에서 `new` 키워드 → GC Alloc 유발
- `Update()`에서 문자열 연산 (`+` 연결) → `StringBuilder` 사용
- `renderer.material` Update()에서 반복 접근 → Awake()에서 캐싱 (`sharedMaterial`은 전체 공유 변경 시만)
- `public` 필드 직접 노출 → `[SerializeField] private` 사용
- 구형 `Input.GetKey()` → New Input System 사용
- `Resources.Load()` → Addressables로 대체 (런타임 메모리 관리 불가)
- `OnDisable/OnDestroy`에서 이벤트 구독 해제 누락

---

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
public event Action<float> OnHealthChanged;

private void OnEnable()  { GameEvents.OnStageCleared += HandleStageCleared; }
private void OnDisable() { GameEvents.OnStageCleared -= HandleStageCleared; } // 반드시 해제
```

---

## 비동기 처리

```csharp
// Awaitable (Unity 6 권장 — 메인 스레드 유지)
public async Awaitable DelayedAction(float seconds, CancellationToken ct)
{
    await Awaitable.WaitForSecondsAsync(seconds, ct);
    Execute();
}
// 씬 로드 등 긴 작업: Coroutine + LoadSceneAsync
```

---

## SOLID 원칙 (Unity 맥락)

패턴 코드 상세는 unity-patterns 스킬 참조.

| 원칙 | Unity 핵심 적용 |
|------|----------------|
| SRP | Player → PlayerInput, PlayerMovement, PlayerAudio 분리 |
| OCP | 추상 클래스로 확장, 기존 코드 수정 불필요 |
| LSP | 서브 클래스가 기본 클래스로 완전 대체 가능해야 함 |
| ISP | IMovable, IDamageable 등 작고 집중된 인터페이스 분리 |
| DIP | 인터페이스로 추상화, 구상 클래스에 직접 의존 금지 |

---

## 사용 패키지 기준
- 에셋 관리: Addressables
- 카메라: Cinemachine
- 입력: Input System (New)
- 비동기: Coroutine 또는 Awaitable (프로젝트별 선택)

---

## 오브젝트 풀링 (UnityEngine.Pool)

반복 생성·삭제 오브젝트는 `ObjectPool<T>` 필수 (Unity 6 내장):

```csharp
using UnityEngine.Pool;

private ObjectPool<Bullet> m_pool;

private void Awake()
{
    m_pool = new ObjectPool<Bullet>(
        createFunc:      () => Instantiate(m_bulletPrefab),
        actionOnGet:     b  => b.gameObject.SetActive(true),
        actionOnRelease: b  => b.gameObject.SetActive(false),
        actionOnDestroy: b  => Destroy(b.gameObject),
        defaultCapacity: 10, maxSize: 50
    );
}
```

---

## 성능 기준 (모바일)
- Draw Call: 100 이하 / GC Alloc per frame: 0 목표 / 프레임 예산: 16.6ms (60fps)
- 텍스처: ASTC 압축, Atlas 사용 / 반복 생성 객체: ObjectPool 필수
