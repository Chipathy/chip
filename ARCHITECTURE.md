# PM Agent 全流程风险副驾驶 Architecture v1.1

## 1. 架构目标

本系统是面向芯片项目 PM 的全流程风险副驾驶（PM Risk Copilot）。

供应链延期风险是 MVP 的首个落地场景，但架构需要支持后续扩展研发、测试、质量和生产等风险领域。

核心变化：

- PM 是风险接收、确认和决策的人，不是系统唯一的风险输入者。
- Risk Copilot 主动分析项目多源信息并发现潜在风险。
- 飞书 Aily 是 Agent 的主要运行环境和用户交互入口。
- Risk Agent 负责统一的风险分析，Rule、Case Knowledge Base（RAG）和 Impact Analysis 是其调用能力。
- FastAPI 不再作为整个 Agent 系统的核心，仅作为可选辅助数据服务层。

## 2. 总体架构

```text
                         飞书 Aily
                             │
                    PM Risk Copilot
                             │
                             ↓
                        Risk Agent
                             │
             ┌───────────────┼───────────────┐
             │               │               │
           Rule     Case Knowledge Base（RAG）  Impact Analysis
             └───────────────┼───────────────┘
                             ↓
                         Risk Card
                             ↓
                   Collaboration Agent
```

### 2.1 飞书 Aily 定位

飞书 Aily 是 MVP 阶段的主要 Agent 运行环境，负责：

- 承载 PM Risk Copilot
- 接收和分析项目相关信息
- 调用 Risk Agent 能力
- 组织 Rule、Case Knowledge Base（RAG）和影响分析结果
- 生成并推送风险卡片
- 接收 PM 的确认和后续指令

飞书承担 Agent 运行、知识使用和消息交互能力，不代表 PM 必须手动输入风险后系统才能工作。MVP 数据默认使用飞书 Aily 内部能力。

## 3. 核心数据流

```text
      项目数据 / 项目状态变化 / 团队反馈 / 业务事件
                         ↓
                   Event 标准化
                         ↓
                  项目相关信息输入
                         ↓
                     Event标准化
                         ↓
               Risk Agent启动风险分析
                         ↓
                    调用分析能力：
                 ├── Rule规则判断
                 │
                 ├── Case Knowledge Base（RAG）检索历史案例
                 │
                 └── Impact Analysis分析影响
                          ↓
                   Risk Agent综合判断
                          ↓
                    生成Risk Card
                          ↓
                        PM确认
                          ↓
   Collaboration Agent执行风险通知/协同
                  
```

MVP 阶段使用模拟项目数据验证从风险发现、分析、Risk Card 推送到 PM 确认和协同通知的完整链路。

系统主动分析的信息可以包括：

- 项目状态变化
- 异常事件
- 团队反馈
- 风险相关文档信息

具体数据接入方式由飞书 Aily 内部能力定义。

## 4. Risk Agent 及其调用能力

### 4.1 Risk Agent

Risk Agent作为全流程风险分析的核心编排模块。接收到标准化Event后，根据风险场景调用Rule规则能力、Case Knowledge Base（RAG）以及Impact Analysis能力，综合生成最终Risk结果。Rule和Case Knowledge Base不是独立Agent。

职责：

1. 理解项目风险事件。
2. 识别异常和潜在风险。
3. 调用规则能力判断风险等级。
4. 调用 Case Knowledge Base（RAG）获取历史经验。
5. 调用 Impact Analysis 分析影响范围。
6. 生成结构化 Risk 和 Risk Card。
7. 给出处理建议。

风险来源示例：

| 风险领域 | `event_type` 示例 |
|---|---|
| 供应链 | `supplier_delay` |
| 研发 | `development_delay` |
| 测试 | `test_failure` |
| 质量 | `quality_issue` |

MVP 优先实现 `supplier_delay`。

### 4.2 Rule 规则能力

Rule 是 Risk Agent 使用的确定性判断依据，用于辅助判断风险等级和建议动作。

MVP 阶段：

- `threshold` 以字符串形式保存规则描述。
- 规则由 Aily 固定逻辑或 Risk Agent 按约定解释。
- 不实现通用规则 DSL 或复杂规则执行引擎。

示例：

```text
delay_days>=14 → High
```

### 4.3 Case Knowledge Base（RAG）

Case 是 Risk Agent 使用的历史经验知识，不是独立 Agent。

Case Knowledge Base（RAG）负责：

- 根据当前风险检索相似历史案例
- 提供问题、原因、影响、方案和结果参考
- 辅助 Risk Agent 生成可解释的判断和建议

MVP 可以使用 JSON Case 库和基础文本/关键词检索；Embedding 检索作为可选增强，不要求建设大规模向量数据库。

### 4.4 Impact Analysis Module

Impact Analysis Module 是风险影响分析模块，不是 Decision Agent，也不自动替代 PM 决策。

职责：

- 识别受影响项目节点
- 分析影响范围
- 判断简单延期影响
- 输出建议动作

MVP 不实现：

- 自动优化方案
- 复杂仿真
- 成本模型
- 资源优化

### 4.5 Collaboration Agent

Collaboration Agent 是 Risk 输出后的执行能力，负责风险确认后的协同动作：

- 推送风险通知
- 通知相关负责人
- 执行风险通知和确认闭环

MVP 只实现风险通知和确认闭环。任务创建和复杂跟进能力作为未来扩展。

## 5. 辅助数据服务层（可选）

当前Demo优先基于飞书Aily内部能力实现，FastAPI仅作为未来企业系统数据接入时的可选扩展。

### 5.1 可选职责

- 读取 Mock 数据
- 提供数据格式转换
- 为 Aily 提供必要的数据读取或持久化辅助
- 对接外部信息来源

FastAPI 不负责：

- 作为整个 Agent 的运行环境
- 取代飞书 Aily 的 Agent 编排
- 独立承担 Risk Agent 的全部分析逻辑

## 6. 数据存储策略

### 6.1 MVP 默认使用 Aily 内部能力

- Aily Agent 运行能力
- Aily 知识库保存 Case
- Aily 数据能力保存项目数据、Event 和 Risk 记录
- 飞书卡片完成风险展示和 PM 交互

所有 MVP 业务数据默认使用飞书 Aily 内部能力管理，不设置外部数据库路线。

### 6.2 本地数据

Mock JSON 仅用于初始化数据和 Demo 准备，不作为独立的运行时数据存储：

```text
data/
├── mock/
│   ├── projects.json
│   ├── milestones.json
│   ├── events.json
│   ├── rules.json
│   └── cases.json
└── output/
    └── risks.json
```

## 7. 数据模型摘要

核心对象保持不变：

`projects`、`milestones`、`events`、`rules`、`cases`、`risks`

具体字段以《数据模型设计.md》为准。

### Project

项目基础信息，包括项目名称、阶段、负责人和目标日期。

### Milestone

项目关键节点，用于识别风险影响对象。

### Event

项目风险事件，只描述客观事实。主要字段包括：

`event_id`、`project_id`、`milestone_id`、`risk_domain`、`source_department`、`event_type`、`source`、`supplier`、`material`、`delay_days`、`description`、`occurred_at`、`extra_data`

### Rule

风险判断依据，包括：

`rule_id`、`rule_name`、`risk_domain`、`threshold`、`risk_level`、`suggest_action`

### Case

历史经验案例，包括：

`case_id`、`title`、`category`、`domain`、`context`、`problem`、`cause`、`impact`、`solution`、`result`、`risk_pattern`、`trigger_conditions`、`recommendation`、`tags`

### Risk

Risk Agent 分析后的结构化风险结果，用于风险卡片、历史记录和 PM 审核闭环。

## 8. 项目目录建议

```text
PM-Risk-Copilot/
├── docs/
├── data/
│   ├── mock/
│   └── output/
├── prompts/
├── knowledge_base/
├── services/
└── README.md
```

目录原则：

- Prompt 独立保存
- Case 知识库独立管理
- Mock 数据与 Risk 输出分离
- 可选数据服务放在 `services/`
- 不把业务数据写死在 Agent 逻辑中

## 9. 飞书 Risk Card 与 PM 交互

Risk Card 至少包含：

- 项目和受影响节点
- 风险标题
- 风险等级
- 风险原因
- 触发事件
- 命中规则
- 相似历史案例
- 影响分析
- 建议动作

交互流程：

```text
Risk Copilot 发现风险
        ↓
生成 Risk Card
        ↓
飞书主动推送 PM
        ↓
PM 查看原因、影响和建议
        ↓
PM 确认或驳回风险
        ↓
Collaboration Agent 完成风险通知和确认闭环
```

PM 保留最终确认和决策权，AI 不自动替代 PM 做最终判断。

## 10. MVP 边界

### 10.1 必须完成

- 飞书 Aily 中运行 PM Risk Copilot
- 从项目相关信息中发现供应链延期异常
- Risk Agent 生成 High 风险判断
- Rule 辅助风险判断
- Case Knowledge Base（RAG）返回相似案例
- Impact Analysis 输出影响节点、影响范围和建议动作
- 生成并推送 Risk Card
- PM 确认风险
- 风险通知和确认闭环

### 10.2 暂不实现

- 完整自动决策
- 自动优化方案
- 复杂仿真
- 成本模型
- 资源优化
- 任务创建和复杂跟进（作为未来扩展）
- 全领域风险覆盖
- 大规模向量数据库
- 实时 ERP/MES 全量接入

## 11. 最终 Demo 流程

```text
项目相关数据产生：BGA供应商延期15天
              ↓
Risk Copilot 主动发现异常
              ↓
Risk Agent 分析
              ↓
Rule 判断为 High 风险
              ↓
Case Knowledge Base（RAG）匹配 CASE001
              ↓
Impact Analysis 分析封测节点影响
              ↓
生成 Risk Card
              ↓
飞书主动推送 PM
              ↓
PM 确认风险
              ↓
Collaboration Agent 完成风险通知和确认闭环
```

核心验收链路：

```text
多源项目风险信息 → 主动发现 → Rule → Case Knowledge Base（RAG） → Impact Analysis → Risk Card → PM 确认 → 协同通知
```
