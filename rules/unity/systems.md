# Unity 시스템 규칙 — 오디오 · 다국어 · 접근성 · 학습

## 오디오
- AudioMixer 계층 구조 사용 (BGM / SFX / Voice / UI)
- 반복 재생 SFX → 오브젝트 풀링 적용
- 볼륨: dB 단위, 로그 스케일 변환 주의

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
