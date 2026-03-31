# Unity 코드 리뷰 상세 체크리스트

## 성능 (Performance)
| 항목 | 심각도 |
|------|--------|
| Update/FixedUpdate/LateUpdate에서 GetComponent, Find 계열 호출 | 🔴 |
| 매 프레임 new 키워드 → GC Alloc | 🔴 |
| 런타임 루프에서 LINQ 사용 | 🟡 |
| 런타임 루프에서 string 연결 (+) → StringBuilder 필요 | 🟡 |
| 반복 생성·파괴 객체에 ObjectPool 미적용 | 🟡 |

## Unity 규칙
| 항목 | 심각도 |
|------|--------|
| public 필드 노출 (SerializeField 미사용) | 🟡 |
| Magic number (상수화 필요) | 🟡 |
| Resources.Load() 사용 → Addressables 필요 | 🟡 |
| 구형 Input.GetKey() 사용 | 🟡 |
| OnDisable/OnDestroy에서 이벤트 구독 해제 누락 | 🔴 |
| Coroutine → UniTask 대체 가능 여부 | 🟢 |

## 아키텍처
| 항목 | 심각도 |
|------|--------|
| 단일 책임 원칙 위반 | 🟡 |
| 싱글턴 남용 | 🟡 |
| 강한 결합 (이벤트/인터페이스 분리 가능) | 🟡 |
| 메서드 30줄 초과 | 🟢 |

## 안전성
| 항목 | 심각도 |
|------|--------|
| null 참조 가능성 (TryGet 패턴 미사용) | 🔴 |
| 씬 전환 시 참조 유실 가능성 | 🔴 |
| Destroy 후 참조 접근 가능성 | 🔴 |
| 비동기 CancellationToken 미전달 | 🟡 |
