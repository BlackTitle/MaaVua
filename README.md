# MaaVua &middot; ![Version](https://img.shields.io/badge/version-v0.1.0-blue) ![License](https://img.shields.io/badge/license-AGPL--3.0-green) ![Platform](https://img.shields.io/badge/platform-Windows-blue)

基于 [MaaFramework](https://github.com/MaaXYZ/MaaFramework) 的**《热血海贼王》**自动化工具，使用图像识别与 OCR 实现日常任务托管。

> 运行时需搭配 [MFAAvalonia](https://github.com/MaaXYZ/MFAAvalonia) GUI 面板。

## 功能

| 任务 | 说明 |
|------|------|
| 自动推图 | OCR 识别对战按钮，自动推进主线关卡 |
| 自动刷图 | 可配置次数的副本循环挂机 |
| 世界寻宝 | 免费刷新 3 次，自动搜刮宝物 |
| 世界 BOSS | 庞克巨龙 / 北海巨妖 / 奥兹，三 BOSS 全自动 |
| 扫矿踢狗 | 自定义玩家 ID 扫描，自动占领矿点 |

## 快速开始

### 1. 下载

从 [Releases](../../releases) 下载最新打包，解压到**纯英文路径**（如 `D:\MaaVua`）。

### 2. 安装依赖

首次运行前，右键 `DependencySetup_依赖库安装_win.bat` → **以管理员身份运行**，安装 VC++ 运行库和 .NET 10 运行时。

### 3. 启动

双击 `MFAAvalonia.exe`，在界面左侧选择控制器类型为 **Win32 → Vua**，连接游戏窗口后勾选任务，点击开始。

> 游戏窗口标题需包含 `Vua` 才能被自动识别。

## 项目结构

```
MaaVua/
├── interface.json          # 项目入口配置
├── tasks/                  # 任务定义
├── resource/base/
│   ├── pipeline/           # Pipeline 流程文件
│   ├── image/              # 模板匹配图片
│   └── model/ocr/          # OCR 识别模型
├── docs/                   # 开发文档
└── tools/                  # 校验与辅助脚本
```

## 开发

```bash
pnpm install
pnpm check          # 运行全部校验
pnpm check:maa      # MaaFramework 资源检查
pnpm check:schema   # JSON Schema 校验
pnpm lint           # 项目结构检查
```

Pipeline 编辑推荐使用 [MaaPipelineEditor](https://mpe.codax.site) + LocalBridge 连接本地项目。

## 相关项目

- [MaaFramework](https://github.com/MaaXYZ/MaaFramework) — 图像识别自动化框架
- [MFAAvalonia](https://github.com/MaaXYZ/MFAAvalonia) — Avalonia 桌面 GUI 面板
- [MaaPipelineEditor](https://github.com/kqcoxn/MaaPipelineEditor) — 在线 Pipeline 编辑器

## 许可

[AGPL-3.0-or-later](LICENSE)
