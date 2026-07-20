# MFAAvalonia 打包错误分析：`请删除整个文件夹重新下载，或联系项目维护者`

> 仓库：[MaaXYZ/MFAAvalonia](https://github.com/MaaXYZ/MFAAvalonia)
> 分析版本：v2.12.2
> 分析日期：2026-07-20

## 错误触发链

MFAAvalonia 启动流程（`MFAAvalonia.Desktop/Program.cs:Main()`）：

```
Program.Main()
  ├─ CheckSkiaAvailability()          // 检测 SkiaSharp 图形库
  ├─ CleanupDuplicateLibraries()      // ⚠️ 清理重复文件
  ├─ SetupNativeLibraryResolver()     // 设置 DLL 解析器
  ├─ AppRuntime.Initialize()          // 初始化运行时
  └─ BuildAvaloniaApp().Start...()    // 启动 Avalonia UI
       └─ App.Initialize()
            ├─ 初始化日志/语言/配置
            └─ OnFrameworkInitializationCompleted()
                 └─ MaaProcessorManager.LoadInstanceConfig()
                      └─ 加载 interface.json / tasks / pipelines
                           └─ 失败 → LoadResourcesFailed → 弹出错误
```

## 两类触发场景

### 场景一：MaaAgentBinary 重复目录清理失败

**代码位置**: `MFAAvalonia/PrivatePathHelper.cs:CleanupDuplicateLibraries()`

```csharp
var agentPath = Path.Combine(baseDirectory, "MaaAgentBinary");
if (Path.Exists(agentPath))
    Directory.Delete(agentPath, true);
```

**根因**: GitHub 发布的 zip 包中，`MaaAgentBinary/` 同时存在于根目录和 `libs/` 子目录（各 78 个文件）。首次启动时程序会删除根目录下的重复副本。如果删除失败（文件被锁定、权限不足、上次崩溃残留），异常在 catch 块被吞掉（log warning），但如果后续资源加载也失败，会弹出错误对话框。

**典型表现**:
- 从压缩包直接拖拽到受保护目录（如 `Program Files`、`Downloads` 被安全软件锁定）
- 上次运行被强制终止，文件处于不一致状态
- 文件携带只读属性（从 zip 解压后可能带有）

### 场景二：MaaVua 项目资源加载失败

**代码位置**: `MFAAvalonia/App.axaml.cs` → `LoadResourcesFailed` → `LoadResourcesFailedDetail`

```
LoadResourcesFailed       = "资源加载失败！"
LoadResourcesFailedDetail = "请删除整个文件夹重新下载，或联系项目维护者。"
```

**根因**: MFAAvalonia 加载项目资源（`interface.json` + `tasks/` + `resource/`）时，如果 JSON 文件格式错误、缺少必要字段、或 Pipeline 引用了不存在的资源，会触发此错误。

**本项目常见原因**:
- `interface.json` 中 `import` 路径指向不存在的文件
- Pipeline JSON 语法错误（如缺少逗号、引号不匹配）
- `resource/base/` 缺少 `model/ocr/` 下的 OCR 模型文件
- `config/` 目录下存在格式错误的配置文件

## 诊断方法

### 1. 检查日志
首次运行后查看生成的日志文件：
- `appsettings.json` — 实例配置
- `config/config.json` — 全局配置
- `logs/log-yyyymmdd.log` — 运行日志
- `debug/maafw.log` — MaaFramework 日志

### 2. 验证 JSON 格式
```powershell
# 在项目根目录运行
pnpm check
```

### 3. 手动排查文件完整性
检查打包目录是否包含以下关键文件：
```
MFAAvalonia.exe
libs/MaaFramework.Binding.dll
runtimes/win-x64/native/MaaFramework.dll
interface.json
tasks/vua_main.json
resource/base/
  ├─ default_pipeline.json
  ├─ image/         (50 个 .png)
  ├─ model/ocr/     (det.onnx, rec.onnx, keys.txt)
  └─ pipeline/
       ├─ vua_main.json
       ├─ AutoPang.json
       ├─ DealyBoss.json
       └─ WorldTreasure.json
```

## 打包注意事项

1. **首次启动**: `libs/MaaAgentBinary` 和根目录 `MaaAgentBinary` 重复是正常的，程序会自动清理。不要手动删除 libs/ 下的。
2. **资源同步**: 务必使用 `robocopy /MIR` 将项目最新资源同步到打包目录
3. **只读属性**: 从 zip 解压后，使用以下命令清除只读属性：
   ```powershell
   Get-ChildItem -Recurse | ForEach-Object { $_.Attributes = $_.Attributes -band -bnot [System.IO.FileAttributes]::ReadOnly }
   ```
4. **工作目录**: 运行 `MFAAvalonia.exe` 时必须以其所在目录为工作目录，否则 `SubdirectoriesToProbe` 路径解析会失败
5. **代理设置**: 如果开启了全局代理，可能影响资源更新检测，建议关闭

## 常见误区

- ❌ 从 zip 双击打开后直接运行（实际运行在临时目录，程序会检测并提示）
- ❌ 只复制 `MFAAvalonia.exe` 不复制 `libs/` 和 `runtimes/`
- ❌ 将打包目录放在 OneDrive/同步文件夹中（文件锁导致清理失败）
- ❌ 同时运行多个 MFAAvalonia 实例（MaaAgentBinary 目录被锁定）
