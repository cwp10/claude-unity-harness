# Unity 성능 & 설계 원칙

## 성능 기준 (모바일)
- Draw Call: 100 이하 / GC Alloc per frame: 0 목표 / 프레임 예산: 16.6ms (60fps)
- 텍스처: ASTC 압축, Atlas 사용 / 반복 생성 객체: ObjectPool 필수
- 런타임 루프(Update 등) 안에서 LINQ 사용 금지 — GC Alloc 유발 (🟡 Warning)

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
