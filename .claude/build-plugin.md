# 플러그인 빌드 방법

## 빌드 스크립트 (PowerShell)

```powershell
cd claude-unity-harness

# 기존 파일 제거
Remove-Item -Force ..\claude-unity-harness.plugin -ErrorAction SilentlyContinue
Remove-Item -Force ..\claude-unity-harness-temp.zip -ErrorAction SilentlyContinue

# 압축 후 .plugin 확장자로 변경
Compress-Archive -Path * -DestinationPath ..\claude-unity-harness-temp.zip
Rename-Item ..\claude-unity-harness-temp.zip ..\claude-unity-harness.plugin
```

## GitHub Release 업로드

1. [GitHub Releases](https://github.com/cwp10/claude-unity-harness/releases) 에서 새 릴리즈 생성
2. `claude-unity-harness.plugin` 파일 첨부
3. **"Set as latest release"** 체크 후 게시
4. 설치 URL (태그 무관, 항상 최신):
   ```
   https://github.com/cwp10/claude-unity-harness/releases/latest/download/claude-unity-harness.plugin
   ```
