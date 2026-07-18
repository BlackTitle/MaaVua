# 任务状态调度流程

开始节点是最高优先级调度器。所有操作完成后都返回开始节点，由内部任务状态决定下一步动作。

任务栏持续可见只代表可以读取任务信息，不应直接触发重复接取。接取成功后，任务状态切换为“执行中”；在到达下一次任务检查时间前，开始节点直接调度打怪模块，避免任务扫描长期占用执行流程。

```mermaid
flowchart TD
    START([开始节点<br/>最高优先级调度器])

    START --> STATE{当前内部任务状态}

    STATE -->|待提交| SUBMIT[提交任务]
    STATE -->|正在接取| CONFIRM[确认是否接取成功]
    STATE -->|执行中| CHECK_TIME{是否到达<br/>任务状态检查时间}
    STATE -->|无任务| SCAN_NEW[扫描可接取任务]

    SUBMIT --> SUBMIT_RESULT{提交结果}
    SUBMIT_RESULT -->|成功| RESET[清除当前任务<br/>状态设为无任务]
    SUBMIT_RESULT -->|失败且可重试| WAIT_SUBMIT[等待后重试]
    SUBMIT_RESULT -->|超过重试次数| RESYNC[强制重新同步状态]

    RESET --> START
    WAIT_SUBMIT --> START
    RESYNC --> START

    CONFIRM --> ACCEPT_RESULT{接取结果}
    ACCEPT_RESULT -->|接取成功| SET_EXECUTING[保存任务指纹<br/>状态设为执行中]
    ACCEPT_RESULT -->|仍在处理中| WAIT_ACCEPT[短暂等待]
    ACCEPT_RESULT -->|接取失败| SET_IDLE[状态设为无任务]

    SET_EXECUTING --> START
    WAIT_ACCEPT --> START
    SET_IDLE --> START

    CHECK_TIME -->|否| BATTLE[进入打怪模块]
    CHECK_TIME -->|是| SCAN_PROGRESS[扫描任务进度]

    SCAN_PROGRESS --> PROGRESS_RESULT{任务状态}
    PROGRESS_RESULT -->|已完成| SET_SUBMIT[状态设为待提交]
    PROGRESS_RESULT -->|仍在进行| UPDATE_TIME[更新下次检查时间]
    PROGRESS_RESULT -->|识别失败| FAIL_COUNT{是否超过<br/>识别重试次数}
    PROGRESS_RESULT -->|任务已消失| RESYNC

    FAIL_COUNT -->|否| UPDATE_TIME
    FAIL_COUNT -->|是| RESYNC

    UPDATE_TIME --> BATTLE
    SET_SUBMIT --> START

    BATTLE --> BATTLE_RESULT{本轮执行结果}
    BATTLE_RESULT -->|正常结束| START
    BATTLE_RESULT -->|目标暂不可用| WAIT_BATTLE[等待或移动]
    BATTLE_RESULT -->|异常| RESYNC
    WAIT_BATTLE --> START

    SCAN_NEW --> NEW_RESULT{是否发现新任务}
    NEW_RESULT -->|是| DUPLICATE{是否为已经处理的<br/>同一个任务}
    NEW_RESULT -->|否| IDLE_ACTION[等待或执行默认行为]
    NEW_RESULT -->|识别失败| IDLE_ACTION

    DUPLICATE -->|否| ACCEPT[执行接取任务<br/>状态设为正在接取]
    DUPLICATE -->|是且仍在执行| SET_EXECUTING
    DUPLICATE -->|是但状态异常| RESYNC

    ACCEPT --> START
    IDLE_ACTION --> START

    classDef start fill:#2563eb,color:#fff,stroke:#93c5fd,stroke-width:2px;
    classDef action fill:#1f2937,color:#fff,stroke:#9ca3af;
    classDef decision fill:#78350f,color:#fff,stroke:#fbbf24;
    classDef state fill:#14532d,color:#fff,stroke:#86efac;

    class START start;
    class STATE,CHECK_TIME,SUBMIT_RESULT,ACCEPT_RESULT,PROGRESS_RESULT,FAIL_COUNT,BATTLE_RESULT,NEW_RESULT,DUPLICATE decision;
    class SUBMIT,CONFIRM,SCAN_NEW,SCAN_PROGRESS,BATTLE,WAIT_SUBMIT,WAIT_ACCEPT,WAIT_BATTLE,IDLE_ACTION,ACCEPT action;
    class RESET,RESYNC,SET_EXECUTING,SET_IDLE,SET_SUBMIT,UPDATE_TIME state;
```

## 状态定义

| 状态 | 含义 |
| --- | --- |
| 无任务 | 当前没有已接取任务，可以扫描任务列表 |
| 正在接取 | 已发出接取操作，等待确认结果 |
| 执行中 | 任务已接取，按任务目标调度打怪等执行模块 |
| 待提交 | 任务目标已经完成，优先执行提交操作 |

## 调度约束

1. 只有“无任务”状态可以扫描可接取任务。
2. “执行中”状态优先进入任务执行模块，仅按检查间隔扫描任务进度。
3. 任务扫描必须返回明确结果，不能从扫描节点直接循环回扫描节点。
4. 接取、提交和识别失败必须限制重试次数，超过限制后重新同步状态。
5. 每个动作结束后返回开始节点，不在模块之间直接跳转。
