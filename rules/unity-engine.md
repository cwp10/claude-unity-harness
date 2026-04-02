---
paths:
  - "Assets/**/*.cs"
---

# Unity 스크립트 규칙

## 버전 기준
- Unity 6 LTS (6000.0.x) / C# 10 / .NET Standard 2.1
- Render Pipeline: 프로젝트별 오버라이드 (기본 URP)

---

## 네이밍 접두사

| 대상 | 접두사 | 예시 |
|------|--------|------|
| private 멤버 변수 | `m_` | `m_currentHealth`, `m_movementSpeed` |
| 상수 | `k_` | `k_MaxItems`, `k_DefaultSpeed` |
| 정적 변수 | `s_` | `s_instance`, `s_count` |

이벤트 발생 메서드: `On` 접두사 / 관찰자 처리: `주체이름_이벤트이름`

### 에셋 네이밍
| 에셋 | 접두사 | 에셋 | 접두사 |
|------|--------|------|--------|
| 씬 | `Scene_` | 프리팹 | `PFB_` |
| ScriptableObject | `SO_` | 애니메이션 | `ANIM_` |
| 머티리얼 | `MAT_` | 텍스처 | `TEX_` |

---

## 생명주기 순서

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
- `Update()`에서 문자열 연산 (`+`) → `StringBuilder` 사용
- `renderer.material` Update()에서 반복 접근 → Awake()에서 캐싱
- `public` 필드 직접 노출 → `[SerializeField] private` 사용
- 구형 `Input.GetKey()` → New Input System 사용
- `Resources.Load()` → Addressables로 대체
- `OnDisable/OnDestroy`에서 이벤트 구독 해제 누락

---

## 사용 패키지
- 에셋: Addressables / 카메라: Cinemachine / 입력: Input System (New)
- 비동기: Coroutine 또는 Awaitable (프로젝트별)
