# MaaPipelineEditor LocalBridge 部署与使用笔记

> 来源：[MaaPipelineEditor 本地服务：概览与部署](https://mpe.codax.site/docs/guide/server/deploy.html)  
> 整理日期：2026-07-18。上游工具可能继续更新，实际使用前可对照原文和 `mpelb --help`。

## 用途

MaaPipelineEditor（MPE）本身可以直接在线使用。LocalBridge（命令名 `mpelb`）是可选的本地增强服务，用 WebSocket 把在线编辑器与本地项目连接起来，主要提供：

- 扫描、打开、保存、新建和监听本地 `.json`、`.jsonc` Pipeline 文件
- 设备连接、截图、OCR、取色和区域选择
- Pipeline 流程调试、节点级运行、追踪和产物查看
- AI 节点搜索、补全及 MaaMCP 支持

当前推荐使用 **Web + LocalBridge**。Desktop 一体包暂时下线。

## 安装

### Windows PowerShell（推荐）

```powershell
irm https://raw.githubusercontent.com/kqcoxn/MaaPipelineEditor/main/scripts/install/install.ps1 | iex
```

该脚本会安装或更新 `mpelb`，并准备 MaaFramework runtime 和原生 OCR 资源。安装完成后，新开的终端中应能直接使用：

```powershell
mpelb --help
mpelb info
```

通过网络下载并直接执行脚本前，应确认 URL 和脚本来源可信；需要审查时，可先单独下载并阅读脚本再执行。

### Linux/macOS

```bash
curl -fsSL https://raw.githubusercontent.com/kqcoxn/MaaPipelineEditor/main/scripts/install/install.sh | bash
```

### GitHub API 限流

安装出现“获取版本信息失败”时，通常是 GitHub 未认证 API 的每 IP 每小时 60 次限制。创建一个无需任何 scope 的 GitHub Token 后，在当前 PowerShell 会话中执行：

```powershell
$env:GITHUB_TOKEN="your_github_token"
irm https://raw.githubusercontent.com/kqcoxn/MaaPipelineEditor/main/scripts/install/install.ps1 | iex
```

Token 创建入口：<https://github.com/settings/tokens>。

### 手动安装

也可以从 [MaaPipelineEditor Releases](https://github.com/kqcoxn/MaaPipelineEditor/releases) 下载平台对应文件：

- Windows：`mpelb-windows-amd64.exe`
- Linux：`mpelb-linux-amd64`
- macOS Apple Silicon：`mpelb-darwin-arm64`

手动下载只包含 LocalBridge 本体，不包含 MaaFramework runtime 和 OCR 资源。需要截图、OCR、设备连接或调试功能时，应自行准备运行环境，或配置已有环境路径。

## 本项目启动方式

LocalBridge 会递归扫描根目录。根目录应尽量接近 Pipeline 文件，严禁在 `C:\` 等过大的上层目录启动，否则可能产生大量递归扫描和读取。

本项目的 Pipeline 位于 `resource/base/pipeline/`，建议在仓库根目录运行：

```powershell
mpelb --root .\resource\base
```

等价方式是先在 `resource/base` 目录打开终端，再直接运行：

```powershell
mpelb
```

默认配置如下：

| 配置 | 默认值 |
| --- | --- |
| 扫描目录 | 当前工作目录 |
| WebSocket 端口 | `9066` |
| 日志级别 | `INFO` |
| 配置位置 | 系统用户数据目录 |

常用自定义启动命令：

```powershell
mpelb --root .\resource\base --port 9066 --log-level INFO --log-dir .\logs
```

调试时可以改为 `--log-level DEBUG`。可用级别为 `DEBUG`、`INFO`、`WARN`、`ERROR`。

## 连接在线编辑器

1. 启动 `mpelb`，确认终端出现 `WebSocket 服务已启动: ws://localhost:9066`。
2. 打开 <https://mpe.codax.site/stable>。
3. 点击右上角 LocalBridge 连接按钮。
4. 已连接时，按钮会显示 `MPE LocalBridge`。
5. 可在“配置 -> 本地通信”中开启自动连接。

如果启动时使用了其他端口，例如 `--port 9999`，必须同时把编辑器“配置 -> 本地通信 -> 端口”改为 `9999`，然后重新连接。

停止使用时，在编辑器中断开连接，或关闭运行 `mpelb` 的终端。服务断开后不会自动重连，需要手动点击连接按钮。

## 日常文件工作流

连接成功后，MPE 的本地文件面板会显示扫描根目录下的 `.json` 和 `.jsonc` 文件。推荐流程：

1. 从本地文件面板打开 `pipeline/vua_main.json`。
2. 在 MPE 中编辑、连线和调试。
3. 保存前确认当前标签对应的真实文件路径。
4. 使用 JSON 面板的“保存到本地”覆盖原文件。
5. 回到 Git 中检查 diff，确认没有非预期的格式或逻辑变化。

保存会直接覆盖本地文件且无法由 LocalBridge 撤销。重要修改应依赖 Git 或手动备份。

LocalBridge 会监听文件变化。开启“自动同步本地文件更改”后，外部编辑会自动重新加载；关闭时会弹窗询问。多人协作或同时使用外部编辑器时可开启，需人工核对变更时建议关闭。

## MaaFramework 与 OCR 配置

一键安装默认会在 `mpelb` 所在目录准备：

```text
runtime/
├── maafw/
│   └── bin/
└── resource/
    └── model/
        └── ocr/
```

未手动配置时，LocalBridge 自动使用上述 runtime。已有 MaaFramework 环境时，可覆盖路径：

```powershell
mpelb config set-lib "D:\path\to\MaaFramework\bin"
mpelb config set-resource "D:\path\to\resource"
```

其他配置命令：

```powershell
mpelb config open
mpelb info
```

- `config open`：使用系统默认编辑器打开配置文件
- `info`：查看运行模式、配置文件位置和日志目录
- 前端“LocalBridge 配置”对话框保存后支持热重载，一般无需重启服务

手动设置的 MaaFramework/OCR 路径存在时，优先级高于安装目录中的默认 runtime。需要更新默认附属环境时，删除安装目录下整个 `runtime` 或对应子目录，再重新运行安装脚本；安装脚本不会覆盖已有 `config.json`。

## 更新

LocalBridge 启动时会检查新版本。更新命令与安装命令相同：

```powershell
irm https://raw.githubusercontent.com/kqcoxn/MaaPipelineEditor/main/scripts/install/install.ps1 | iex
```

脚本每次都会更新 `mpelb` 本体；已有且非空的 MaaFramework runtime 和 OCR 资源不会重复下载。

## 故障排查

### `mpelb` 命令不存在

- 关闭并重新打开终端，让 PATH 更新生效
- 运行安装脚本确认没有报错
- 手动安装时，确认可执行文件已重命名为 `mpelb.exe` 并加入 PATH

### 编辑器连接失败

- 确认 `mpelb` 终端仍在运行
- 确认日志显示的 WebSocket 端口与编辑器端口一致
- 默认地址应为 `ws://localhost:9066`
- 检查端口占用、防火墙或安全软件拦截
- 修改端口后，重新点击编辑器连接按钮

### 文件列表为空或扫描过慢

- 检查 `--root` 是否指向实际包含 `.json`/`.jsonc` 的目录
- 本项目推荐使用 `--root .\resource\base`
- 不要从磁盘根目录或用户主目录启动服务
- 在本地文件面板手动点击“刷新”触发重新扫描

### MaaFramework/OCR 不可用

- 运行 `mpelb info` 检查实际配置路径
- 使用 `mpelb config set-lib` 指向 MaaFramework Release 的 `bin` 目录
- 使用 `mpelb config set-resource` 指向包含 `model/ocr` 的资源目录
- 检查 OCR 目录中是否存在 `det.onnx`、`rec.onnx`、`keys.txt`

## 命令速查

```powershell
# 本项目推荐启动
mpelb --root .\resource\base

# 自定义端口和调试日志
mpelb --root .\resource\base --port 9999 --log-level DEBUG

# 查看当前环境信息
mpelb info

# 打开配置文件
mpelb config open

# 配置已有 MaaFramework 和资源目录
mpelb config set-lib "D:\path\to\MaaFramework\bin"
mpelb config set-resource "D:\path\to\resource"
```
