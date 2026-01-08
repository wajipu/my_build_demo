# CI 工作流（Android / Windows / iOS）

本项目使用 GitHub Actions 进行多平台构建产物。

## 工作流与产物
- Android：`.github/workflows/android-build.yml`
  - 触发：手动（`workflow_dispatch`）
  - 产物：`build/app/outputs/flutter-apk/app-release.apk`
- Windows：`.github/workflows/windows-build.yml`
  - 触发：手动（`workflow_dispatch`）
  - 产物：`build/windows/**/runner/Release/**`
- iOS：`.github/workflows/ios-build.yml`
  - 触发：push / PR / 手动（当前启用）
  - 产物：`build/ios/ipa/*.ipa`

## iOS 签名（Ad Hoc 直装 IPA）
该流程用于生成可直接安装的 IPA，必须使用 **Ad Hoc**
描述文件和 Distribution 证书。

需要在 GitHub Secrets 配置：
- `IOS_P12_BASE64`（包含私钥的 `.p12` 的 base64）
- `IOS_P12_PASSWORD`
- `IOS_MOBILEPROVISION_BASE64`（Ad Hoc `.mobileprovision` 的 base64）
- `IOS_TEAM_ID`
- `IOS_BUNDLE_ID`（必须与描述文件一致）
- `IOS_PROFILE_NAME`（描述文件的 Name 字段）

关键注意事项：
- iOS 描述文件必须是 **Ad Hoc**（不能是 App Store 或 Development）。
- 描述文件要包含设备 UDID，否则无法直装。
- Xcode 的 Bundle ID 必须与描述文件一致：
  - 当前为：`cc.zhsoft.zlls.zmanager`
- 签名文件不要提交：`en/` 和 `*.b64` 已被 `.gitignore` 忽略。

## Windows 下生成 base64（PowerShell）
使用绝对路径并复制到剪贴板：
```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("D:\study\my_build_demo\en\zhongliu_pro.p12")) | Set-Clipboard
[Convert]::ToBase64String([IO.File]::ReadAllBytes("D:\study\my_build_demo\en\your_ad_hoc_profile.mobileprovision")) | Set-Clipboard
```

## Artifact 名称带 Flutter 版本号
可以把 Flutter 版本号拼到 `name`。Linux/macOS 示例：
```yaml
- name: 获取 Flutter 版本
  id: flutter_version
  run: |
    FLUTTER_VERSION=$(flutter --version --machine | python3 -c "import json,sys;print(json.load(sys.stdin)['frameworkVersion'])")
    echo "FLUTTER_VERSION=$FLUTTER_VERSION" >> $GITHUB_ENV

- name: 上传产物
  uses: actions/upload-artifact@v4
  with:
    name: app-${{ env.FLUTTER_VERSION }}
    path: build/ios/ipa/*.ipa
```

Windows Runner 示例：
```yaml
- name: 获取 Flutter 版本（Windows）
  shell: pwsh
  run: |
    $v = (flutter --version --machine | ConvertFrom-Json).frameworkVersion
    "FLUTTER_VERSION=$v" | Out-File -FilePath $env:GITHUB_ENV -Append
```
