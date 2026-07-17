# MaaVua 项目指南

本文档旨在帮助开发者（包括 AI）快速了解 MaaVua 项目结构，以便参与项目开发。

## 项目概述

**MaaVua** 是一个基于 MaaFramework 的自动化项目，用于 Vua 相关的自动化任务。项目使用 JSON 配置任务流水线（Pipeline），通过图像识别和模拟控制实现自动化操作。

## 目录结构

```text
MaaVua/
├── resource/                   # 资源目录
│   └── base/                   # 基础资源包
│       ├── pipeline/           # Pipeline JSON 配置文件
│       │   ├── vua_main.json   # 主任务流水线
│       │   └── tutorial.json   # 教程任务流水线
│       ├── model/              # 模型文件
│       │   └── ocr/            # OCR 模型
│       ├── image/              # 图像模板
│       └── default_pipeline.json # 默认 Pipeline 配置
├── tasks/                      # 任务配置目录
│   ├── tutorial.json           # 教程任务配置
│   └── vua_main.json           # 主任务配置
├── tools/                      # 开发工具
│   ├── validate-schema.mjs     # Schema 验证工具
│   ├── check-project.mjs       # 项目检查工具
│   └── schema/                 # JSON Schema 文件
├── config/                     # 配置文件
│   └── maa_pi_config.json      # MaaFW PI 配置
├── interface.json              # 项目接口配置
├── maa-project.json            # MaaFW 项目配置
└── package.json                # Node.js 项目配置
```

## 核心概念

### 术语定义

| 术语                   | 说明                                                           |
| ---------------------- | -------------------------------------------------------------- |
| **Node（节点）**       | Pipeline JSON 中的一个完整对象，包含识别、动作、后继节点等配置 |
| **Task（任务）**       | 若干 Node 按顺序相连的逻辑流程                                 |
| **Entry（入口）**      | Task 中的第一个 Node                                           |
| **Pipeline（流水线）** | pipeline 文件夹中所有 Node 的全体                              |
| **Bundle**             | 标准资源目录结构，包含 pipeline、model、image 等文件夹         |
| **Resource（资源）**   | 多个 Bundle 按顺序加载后的资源结构                             |

### 项目结构说明

1. **resource/base/**: 基础资源包，包含 Pipeline 配置、模型文件和图像模板
2. **tasks/**: 任务配置目录，定义具体的自动化任务
3. **tools/**: 开发工具，用于项目验证和检查
4. **interface.json**: 项目接口配置，定义控制器和资源包
5. **maa-project.json**: MaaFW 项目配置文件

## 开发规范

### 代码风格

- 使用 JSON 配置文件定义 Pipeline
- 遵循 MaaFramework 的 Pipeline 协议规范
- 使用 Prettier 进行代码格式化

### 文件命名规范

- Pipeline 文件使用小写字母和下划线命名
- 任务配置文件使用小写字母和下划线命名
- 模型文件使用描述性名称

### Pipeline 配置规范

1. **节点命名**: 使用有意义的节点名称，如 `Click_Start_Button`
2. **识别算法**: 根据场景选择合适的识别算法（TemplateMatch、OCR 等）
3. **动作配置**: 合理配置点击、滑动等动作参数
4. **超时设置**: 为每个节点设置合理的超时时间

### 开发流程

1. **添加新任务**:
    - 在 `tasks/` 目录创建新的任务配置文件
    - 在 `resource/base/pipeline/` 目录创建对应的 Pipeline 文件
    - 更新 `interface.json` 添加新的任务引用

2. **修改现有任务**:
    - 直接修改对应的 Pipeline JSON 文件
    - 使用 `pnpm check` 验证配置

3. **添加新的识别算法**:
    - 在 Pipeline 节点中配置识别参数
    - 如需自定义识别，使用 Custom 识别类型

### 测试和验证

```bash
# 安装依赖
pnpm install

# 运行所有检查
pnpm check

# 格式化代码
pnpm format

# 仅检查格式
pnpm format:check

# 验证 Schema
pnpm check:schema

# MaaFW 检查
pnpm check:maa
```

### Git 提交规范

- AI 或自动化工具完成文件修改和必要验证后，必须创建 Git 提交，不得只保留未提交改动
- 提交前检查 `git status`、`git diff` 和近期提交记录，只暂存本次任务涉及的文件，不得混入用户或其他任务的改动
- 提交信息遵循 Conventional Commits，格式为 `<type>(<scope>): <description>`；没有合适 scope 时可省略
- 常用 type 包括 `feat`、`fix`、`docs`、`refactor`、`test`、`chore`、`ci`、`build`、`perf` 和 `style`
- description 应简洁、明确并描述实际变更，例如 `chore(repo): 完善忽略规则与提交规范`
- 若验证因本次修改之外的问题失败，可以提交已验证的本次改动，但必须在结果说明中明确记录失败项
- 除非用户明确要求，不得修改、重写或合并已有提交

## 常见开发场景

### 添加新的 Pipeline 节点

1. 在相应的 Pipeline JSON 文件中添加新节点
2. 配置识别算法和动作参数
3. 设置节点属性（next、timeout 等）
4. 使用 `pnpm check` 验证配置

### 修改任务流程

1. 编辑任务配置文件（`tasks/*.json`）
2. 更新入口节点和任务列表
3. 修改 Pipeline 中的节点连接关系

### 添加新的资源包

1. 在 `resource/` 目录创建新的资源包目录
2. 按照 Bundle 结构组织文件（pipeline、model、image）
3. 更新 `interface.json` 添加新的资源包配置

### 调试和日志

- 查看 `maafw.log` 文件了解运行日志
- 使用 MaaDebugger 工具进行调试
- 检查 Pipeline 节点的识别结果和动作执行情况

## 参考资源

- [MaaFramework 文档](https://maafw.com/)
- [Pipeline 协议](https://maafw.com/zh_cn/3.1-任务流水线协议.html)
- [集成文档](https://maafw.com/zh_cn/2.1-集成文档.html)
- [MaaHub 社区](https://hub.maafw.com)

## 工具使用

### validate-schema.mjs

用于验证 Pipeline JSON 文件是否符合 Schema 定义。

### check-project.mjs

用于检查项目结构和配置是否正确。

### maa-tools

MaaFW 官方工具链，提供项目检查和验证功能。

## 注意事项

1. **资源文件**: 确保模型文件和图像模板正确放置在对应目录
2. **路径配置**: 使用相对路径配置资源路径
3. **版本兼容**: 确保与 MaaFramework 版本兼容
4. **性能优化**: 合理设置识别参数以优化性能
5. **错误处理**: 为关键节点配置超时和错误处理机制
