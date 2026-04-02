# C# 패턴 & 주석 규칙

## 주석 규칙
- 코드로 알 수 없는 **이유**만 주석
- public API: `/// <summary>` XML 태그 필수
- 주석 처리한 코드는 삭제 (소스 관리로 이전 코드 확인)
- TODO는 최신 유지, 오래된 것은 삭제
- 헝가리언 표기법 사용 안 함

```csharp
// ❌ 코드가 말해주는 것 반복
Transform targetToShoot; // 표적이 될 대상

// ✅ 이유를 설명
private Rigidbody m_rb; // 성능상 캐싱 필요 — Update에서 GetComponent 호출 금지

/// <summary>
/// 플레이어에게 데미지를 입힙니다. Apply damage to the player.
/// </summary>
public void TakeDamage(float amount) { ... }
```

---

## DRY 패턴

```csharp
// ❌ WET (중복)
private void PlayExplosionA(Vector3 pos) { ... }
private void PlayExplosionB(Vector3 pos) { ... }

// ✅ DRY
private void PlayFX(ParticleSystem particle, AudioClip clip, Vector3 pos) { ... }
```
