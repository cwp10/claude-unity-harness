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
- 씬 간 이벤트 전달에 특히 유용

### 데이터 구조
- 설정·밸런스·파라미터: `SO_Config` 패턴으로 외부화 (기획자·현장 담당자 수정 가능)
- 런타임 상태: 별도 런타임 데이터 컨테이너
- 저장 데이터: JSON 직렬화 + 암호화

---

## UI 아키텍처
- MVP 패턴 (View ↔ Presenter 분리)
- View는 Presenter를 모름 (단방향 의존)
- UI Toolkit 또는 uGUI 프로젝트별 선택
- 큰 버튼·고대비 색상 권장 (터치/마우스 겸용 고려)
- 경고·알람: 색상 + 아이콘 + 소리 3중 표현 (색약 대응)

---

## 물리
- 물리 연산: `FixedUpdate`만 사용, 고정 타임스텝 + `Rigidbody interpolation` 적용
- `Rigidbody` 이동: `MovePosition` / `AddForce` 사용
- `Transform.position` 직접 변경 금지 (물리 충돌 무시됨)

---

## 입력 (New Input System)
- `InputActionAsset` 기반으로 정의
- 입력 처리 전담 클래스 분리 (`PlayerInputHandler`)
- 구형 `Input.GetKey()` 혼용 금지

---

## 오디오
- AudioMixer 계층 구조 사용 (BGM / SFX / Voice / UI)
- 반복 재생 SFX → 오브젝트 풀링 적용
- 볼륨: dB 단위, 로그 스케일 변환 주의

---

## 씬 관리
- 씬 전환: Addressables + LoadSceneAsync
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

---

## 다국어
- Unity Localization 패키지 사용
- 문자열 하드코딩 절대 금지 → LocalizationKey 사용
- 텍스트 확장 여유 확보 (한→영 시 약 30% 길어짐)

---

## 접근성
- UI Scale 옵션 제공
- 자막(CC) 시스템
- 색약 대응 모드 (색상에만 의존하는 UI 금지)
- 키보드/터치 완전 지원

---

## 학습·진행 흐름
- Event-based Step System (단계별 진행)
- 피드백: 점수/힌트 로직은 별도 ScriptableObject
- 학습 이벤트 로깅 (단계 진입, 완료, 소요 시간, 오답)
- Firebase Analytics 또는 자체 로깅 서버 연동
- SCORM/xAPI 연동 필요 시 WebGL 빌드 고려

---

## 빌드 타겟별 고려사항
| 타겟 | 주의사항 |
|------|---------|
| WebGL | 파일 I/O 불가, IndexedDB 활용, 멀티스레드 제한 |
| Android/iOS | 메모리 제한, 발열, 배터리 |
| PC | 해상도 대응, 다중 모니터 |

---

## CI/CD
- 빌드 스크립트: `BuildScript.cs` (Unity Batch Mode)
- 자동화: GitHub Actions 또는 Jenkins
- 빌드 산출물: 날짜+버전 태그 필수
