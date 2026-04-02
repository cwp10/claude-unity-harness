---
paths:
  - "Assets/**/*.cs"
---

# Unity 도메인 패턴

## 핵심 아키텍처 패턴

패턴 코드는 unity-patterns 스킬 참조.

### 상태 관리
- State Machine 패턴 (IState 인터페이스 기반)
- 전환 조건은 StateMachine이 소유, 상태 로직은 State가 소유
- 상태 3개 이상일 때 사용 / 2개 이하는 bool 플래그로 충분

### 이벤트 시스템
- ScriptableObject Event Channel 패턴 사용
- UnityEvent 직접 노출 지양 → SO 이벤트로 분리

### 데이터 구조
- 설정·밸런스: `SO_Config` 패턴으로 외부화
- 런타임 상태: 별도 런타임 데이터 컨테이너
- 저장 데이터: JSON 직렬화 + 암호화

---

## UI 아키텍처
- MVP 패턴 (View ↔ Presenter 분리, View는 Presenter를 모름)
- UI Toolkit 또는 uGUI 프로젝트별 선택
- 경고·알람: 색상 + 아이콘 + 소리 3중 표현 (색약 대응)

---

## 물리
- 물리 연산: `FixedUpdate`만 사용 + `Rigidbody interpolation`
- 이동: `MovePosition` / `AddForce` / `Transform.position` 직접 변경 금지

---

## 입력 (New Input System)
- `InputActionAsset` 기반 / 입력 전담 클래스 분리 (`PlayerInputHandler`)

---

## 씬 관리
- 씬 전환: Addressables + `LoadSceneAsync`
- 공통 씬 (Persistent): 매니저 클래스 상주
- 씬별 초기화: `IInitializable` 인터페이스 통일

---

## 저장 시스템
```
SaveData (JSON)
├── PlayerData      ← 스탯, 레벨, 인벤토리
├── ProgressData    ← 클리어 씬, 업적, 학습 진도
└── SettingsData    ← 볼륨, 그래픽, 언어
```
