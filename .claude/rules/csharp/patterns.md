# C# 패턴 & 주석 규칙

## 주석 규칙 — 기본값은 "주석 없음"

자세한 공통 원칙은 `rules/response.md`의 "주석 작성 원칙" 참조. C# 특화 규칙만 아래 정리.

**허용되는 경우만:**

- 코드로 알 수 없는 **Why** (숨은 제약·성능 이유·버그 우회)

**절대 금지 (발견 시 즉시 삭제):**

- 코드가 말해주는 What 반복
- 필드·프로퍼티·메서드명을 한국어로 풀어쓴 주석 (`// 체력`, `// 이동 속도`)
- 주석 처리된 죽은 코드 — git이 기억함
- 오래된 TODO — 최신화 불가하면 삭제
- 섹션 구분용 장식 주석 (`// ===== Unity Lifecycle =====`)
- private 필드 위의 중복 `<summary>` 블록
- 헝가리언 표기법 및 그에 대한 설명 주석

```csharp
// ❌ 코드가 말해주는 것 반복 — 제거할 것
Transform targetToShoot;            // 표적이 될 대상
private float m_moveSpeed = 5f;     // 이동 속도
private void Awake() { /* 초기화 */ m_rb = GetComponent<Rigidbody>(); }

// ❌ 현재 작업·이슈 언급 — 제거할 것
// issue #42 대응으로 추가됨
// TODO(2023): 나중에 리팩토링
private void Handle() { ... }

// ✅ Why를 설명 — 숨은 제약이 있을 때만
private Rigidbody m_rb; // 성능: Update에서 GetComponent 호출 금지

// ✅ public API XML 문서
/// <summary>
/// 플레이어에게 데미지를 입힙니다. Apply damage to the player.
/// </summary>
public void TakeDamage(float amount) { ... }
```

**판정 기준:** 주석을 지워도 미래 독자가 혼란스럽지 않다면 쓰지 말 것.

---

## DRY 패턴

```csharp
// ❌ WET (중복)
private void PlayExplosionA(Vector3 pos) { ... }
private void PlayExplosionB(Vector3 pos) { ... }

// ✅ DRY
private void PlayFX(ParticleSystem particle, AudioClip clip, Vector3 pos) { ... }
```
