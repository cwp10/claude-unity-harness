# Unity 플랫폼 & CI/CD

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
