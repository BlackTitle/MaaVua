# MaaVua Pipeline 流程图

## 世界寻宝流程

```mermaid
flowchart TD
    世界寻宝 --> 世界寻宝_大厅图标
    世界寻宝 --> 世界寻宝_返回状态

    世界寻宝_大厅图标["世界寻宝_大厅图标<br/>TM:WorldTreasure.png"] --> 世界寻宝_查询状态

    世界寻宝_查询状态["世界寻宝_查询状态<br/>OCR:立即参与<br/>Click:图标位置"] --> 世界寻宝_寻宝状态查询

    世界寻宝_寻宝状态查询 --> 世界寻宝_寻宝亮
    世界寻宝_寻宝状态查询 --> 世界寻宝_寻宝灰

    世界寻宝_寻宝亮["世界寻宝_寻宝亮<br/>TM:TreasureShiny.png<br/>Click"] --> 世界寻宝_刷新船

    世界寻宝_寻宝灰["世界寻宝_寻宝灰<br/>TM:WorldTeasureGray.png"] --> 世界寻宝_查询状态

    世界寻宝_刷新船["世界寻宝_刷新船<br/>TM:TreasureRefrsh.png<br/>Click×3, 间隔1s"] --> 世界寻宝_开始寻宝

    世界寻宝_开始寻宝["世界寻宝_开始寻宝<br/>TM:TreasureButton.png<br/>Click"] --> 世界寻宝_寻宝灰

    世界寻宝_返回状态["世界寻宝_返回状态<br/>TM:WorldTeasureReturn.png"] --> 世界寻宝_寻宝亮

    subgraph 寻宝次数检测
        世界寻宝_寻宝次数["世界寻宝_寻宝次数<br/>Or(5个OCR)"] -->|成功| 世界寻宝_成功次数
        世界寻宝_寻宝次数 -. on_error .-> 世界寻宝_冷却

        世界寻宝_成功次数["世界寻宝_成功次数<br/>OCR:.*0/5.*"] -->|成功| 世界寻宝_返回
        世界寻宝_成功次数 -. on_error .-> 世界寻宝_冷却
    end

    世界寻宝_返回["世界寻宝_返回<br/>TM:WorldTeasureReturn.png<br/>StopTask"] --> END

    世界寻宝_冷却["世界寻宝_冷却<br/>pre_delay:600s"] --> 世界寻宝_寻宝状态查询

    style 世界寻宝 fill:#f9f,stroke:#333,stroke-width:2px
    style 世界寻宝_返回 fill:#f96,stroke:#333,stroke-width:2px
    style 世界寻宝_冷却 fill:#ff9,stroke:#333,stroke-width:1px
    style 世界寻宝_成功次数 fill:#bbf,stroke:#333,stroke-width:1px
```

```mermaid
flowchart TD
    first["first"] --> 收起活动栏
    first --> 收起状态栏

    收起活动栏 --> 状态判定
    收起状态栏 --> 状态判定

    subgraph 主状态判断
        状态判定["状态判定<br/>（timeout:-1, rate_limit:500）"]
        状态判定无黄色["状态判定无黄色<br/>（无黄色主线分支）"]
    end

    状态判定 --> 任务判断2
    状态判定 --> 跳过对话文本
    状态判定 --> 进入副本
    状态判定 --> 副本攻击
    状态判定 --> 对战结束确认
    状态判定 --> 关闭对战窗口
    状态判定 --> 章节结束
    状态判定 --> 任务判断可以完成
    状态判定 --> 任务判断可以完成2
    状态判定 --> 任务判断可以完成3
    状态判定 --> 任务判断可以完成4
    状态判定 --> 任务对话
    状态判定 --> 结束任务
    状态判定 --> 任务下一步
    状态判定 --> 黄色主线
    状态判定 --> 寻找对战循环

    状态判定无黄色 --> 任务判断2
    状态判定无黄色 --> 跳过对话文本
    状态判定无黄色 --> 进入副本
    状态判定无黄色 --> 副本攻击
    状态判定无黄色 --> 对战结束确认
    状态判定无黄色 --> 关闭对战窗口
    状态判定无黄色 --> 章节结束
    状态判定无黄色 --> 任务判断可以完成
    状态判定无黄色 --> 任务判断可以完成2
    状态判定无黄色 --> 任务判断可以完成3
    状态判定无黄色 --> 任务判断可以完成4
    状态判定无黄色 --> 任务对话
    状态判定无黄色 --> 结束任务
    状态判定无黄色 --> 任务下一步
    状态判定无黄色 --> 寻找对战循环

    subgraph 对战流程
        寻找对战循环["寻找对战循环<br/>timeout:5000"] --> 寻找当前应对战1
        寻找对战循环 --> 寻找当前应对战2
        寻找对战循环 --> 寻找当前应对战3
        寻找对战循环 --> 寻找当前应对战脚下找补1
        寻找对战循环 --> 寻找当前应对战脚下找补2
        寻找对战循环 --> 寻找当前应对战脚下找补3
        寻找对战循环 --> 寻找当前应对战脚下找补4
        寻找对战循环 --> 寻找当前应对战脚下找补5
        寻找对战循环 --> 寻找当前应对战脚下找补6

        寻找当前应对战1 --> 状态判定无黄色
        寻找当前应对战2 --> 状态判定无黄色
        寻找当前应对战3 --> 状态判定无黄色
        寻找当前应对战脚下找补1 --> 状态判定无黄色
        寻找当前应对战脚下找补2 --> 状态判定无黄色
        寻找当前应对战脚下找补3 --> 状态判定无黄色
        寻找当前应对战脚下找补4 --> 状态判定无黄色
        寻找当前应对战脚下找补5 --> 状态判定无黄色
        寻找当前应对战脚下找补6 --> 状态判定无黄色

        寻找对战循环 -. on_error .-> 状态判定无黄色
    end

    subgraph 副本流程
        进入副本["进入副本<br/>OCR:进入副本"] --> 状态判定无黄色
        副本攻击["副本攻击<br/>OCR:攻击"] --> 对战结束确认
        副本攻击 -. on_error .-> 状态判定无黄色
        对战结束确认["对战结束确认<br/>OCR:确定"] --> 关闭对战窗口
        对战结束确认 -. on_error .-> 状态判定无黄色
        关闭对战窗口["关闭对战窗口<br/>TM:CloseMap.png"] --> 状态判定无黄色
        章节结束["章节结束<br/>TM:ChapterEnd.png"] --> 状态判定
    end

    subgraph 黄色主线流程
        黄色主线["黄色主线<br/>TM:YellowMain.png"] --> 任务判断击败
        黄色主线 --> 接取任务
        黄色主线 -. on_error .-> 状态判定无黄色

        接取任务["接取任务<br/>TM:quzhao.png"] --> 任务对话
        接取任务 -. on_error .-> 状态判定无黄色

        任务对话["任务对话<br/>TM:FindCallIcon.png"] --> 任务下一步
        任务对话 -. on_error .-> 状态判定无黄色

        任务下一步["任务下一步<br/>TM:FindNextCall.png"] --> 结束任务

        结束任务["结束任务<br/>TM:CallIcon.png"] --> 任务对话
        结束任务 -. on_error .-> 任务判断击败
    end

    subgraph 任务完成判断
        任务判断可以完成["任务判断可以完成<br/>TM:keyiwancheng.png"] --> 接取任务
        任务判断可以完成 -. on_error .-> 任务判断可以完成2

        任务判断可以完成2["任务判断可以完成2<br/>And(OCR:.*可以.*, OCR:.*完成.*)"] --> 接取任务
        任务判断可以完成2 -. on_error .-> 任务判断可以完成3

        任务判断可以完成3["任务判断可以完成3<br/>And(OCR:.*可以完.*, OCR:.*成.*)"] --> 接取任务
        任务判断可以完成3 -. on_error .-> 状态判定

        任务判断可以完成4["任务判断可以完成4<br/>And(OCR:.*可.*, OCR:.*以完成.*)"] --> 接取任务
        任务判断可以完成4 -. on_error .-> 状态判定
    end

    subgraph Boss 判断
        任务判断2["任务判断2<br/>TM:BossCheck.png"] --> 进入副本
        任务判断2 -. on_error .-> 状态判定

        任务判断击败["任务判断击败<br/>TM:jibai.png"] --> 进入副本
        任务判断击败 -. on_error .-> 状态判定无黄色
    end

    subgraph 对话跳过
        跳过对话文本["跳过对话文本<br/>TM:jumptext.png"] --> 任务对话
        跳过对话文本 --> 黄色主线
        跳过对话文本 --> 状态判定无黄色
    end

    style first fill:#f9f,stroke:#333,stroke-width:2px
    style 状态判定 fill:#bbf,stroke:#333,stroke-width:2px
    style 状态判定无黄色 fill:#bbf,stroke:#333,stroke-width:1px
```

## 节点说明

| 节点 | 识别方式 | 动作 | 说明 |
|------|---------|------|------|
| 收起活动栏 | TM: shouqihuodong.png | Click | 初始化收起活动栏 |
| 收起状态栏 | TM: StatusBarCollapse.png | Click | 初始化收起状态栏 |
| 跳过对话文本 | TM: jumptext.png | Click | 跳过 NPC 对话 |
| 进入副本 | OCR: "进入副本" | Click | 点击进入副本按钮 |
| 副本攻击 | OCR: "攻击" | Click | 攻击怪物 |
| 对战结束确认 | OCR: "确定" | Click | 确认战斗结算 |
| 关闭对战窗口 | TM: CloseMap.png | Click | 关闭战斗结果弹窗 |
| 章节结束 | TM: ChapterEnd.png | Click(target) | 章节通关处理 |
| 黄色主线 | TM: YellowMain.png | DoNothing | 检测黄色主线任务 |
| 接取任务 | TM: quzhao.png | Click | 接取新任务 |
| 任务对话 | TM: FindCallIcon.png | Click | 与 NPC 对话 |
| 任务下一步 | TM: FindNextCall.png | Click | 对话下一步 |
| 结束任务 | TM: CallIcon.png | Click | 完成任务对话 |
| 任务判断2 | TM: BossCheck.png | DoNothing | 检测 BOSS 战 |
| 任务判断击败 | TM: jibai.png | Click | 检测击败任务 |
| 寻找对战循环 | - | - | 寻找可攻击的怪物 |
| 寻找当前应对战1~3 | TM: searchboss{1-3}.png | Click | 模板匹配怪物位置 |
| 寻找当前应对战脚下找补1~6 | TM: OCRFootArray{1-6}.png | Click | 补位匹配怪物位置 |

> TM = TemplateMatch（模板匹配）
