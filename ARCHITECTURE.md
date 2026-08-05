# PM Agent 副驾驶 MVP Architecture v1.0

## 1. 项目目标与设计原则

本项目目标是在 12 天黑客松周期内，实现一个面向芯片项目 PM 的全流程风险副驾驶（PM Risk Copilot）Demo。供应链延期风险是 MVP 阶段的首个落地场景。

核心闭环：

```text
PM 通过飞书输入项目风险事件
        ↓
Risk Agent 结合规则和历史案例进行分析
        ↓
生成 Risk 结果和建议
        ↓
通过飞书风险卡片反馈给 PM
```

MVP 优先保证：

- 风险识别闭环
- AI 分析解释能力
- 历史案例 RAG 能力
- 飞书协同体验

架构支持多来源风险输入，未来可扩展研发、测试、质量和生产风险。

MVP 不追求完整企业级系统，不实现复杂自治、多资源优化或大规模数据治理。

## 2. 总体架构

系统由飞书入口、独立 Agent Backend 和数据存储层组成。

```text
                  用户（PM）
                       ↓
                飞书 Bot / 卡片
                       ↓
                  Webhook / API
                       ↓
              FastAPI Agent Backend
                       ↓
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
    Risk Agent     RAG 模块       Action 模块
        ↓
    Impact Analysis
        ↓
     Data Layer
  JSON / Supabase
```

### 2.1 核心数据流

```text
用户输入项目风险事件
    ↓
飞书 Bot
    ↓
Webhook
    ↓
FastAPI Backend
    ↓
生成 Event
    ↓
查询 Rule
    ↓
Case Retrieval 检索案例
    ↓
Risk Agent 分析
    ↓
生成 Risk JSON
    ↓
保存数据库
    ↓
返回飞书风险卡片
    ↓
PM 确认
    ↓
执行协同动作
```

## 3. 核心模块

### 3.1 Feishu Connector

职责：

- 接收用户消息和异常事件输入
- 调用 Backend API
- 返回风险卡片
- 发送风险通知和协同消息

输入示例：

```text
BGA供应商延期15天
```

风险卡片至少包含：

- 风险等级：High
- 风险原因：BGA 延期超过 14 天
- 历史案例：CASE001
- 建议：启动第二供应商验证

### 3.2 FastAPI Backend

Backend 是系统入口服务，负责：

- API 接口
- 输入数据格式转换
- Event 创建
- Rule、Case 数据读取
- Agent 调用
- Risk 结果保存
- 返回前端或飞书结果

主要接口：

| 方法 | 接口 | 用途 |
|---|---|---|
| POST | `/risk/analyze` | 分析事件并生成 Risk |
| POST | `/event/create` | 创建供应链事件 |
| GET | `/risk/{id}` | 查询风险详情 |

### 3.3 Risk Agent（全流程风险分析 Agent）

Risk Agent 是全流程项目风险分析核心模块，不限定供应链领域。

职责：

1. 理解项目风险事件。
2. 匹配风险规则。
3. 检索历史案例。
4. 分析影响。
5. 输出风险卡片。
6. 给出处理建议。

输入：

- Event
- Project
- Milestone
- Rule
- RAG 检索得到的 Case

处理流程：

```text
事件理解 → 规则匹配 → 案例检索 → 风险判断 → 生成 Risk Card
```

输出：Risk JSON。

### 3.4 Case Retrieval（RAG）

职责：根据当前事件，从历史案例库寻找相似案例。

MVP 实现：

- 使用 JSON Case 库
- 使用关键词或基础文本检索
- Embedding 检索可选
- 支持 Top-K 返回

不要求在 12 天内建设复杂向量数据库。

MVP 输入优先覆盖供应链延期事件 `supplier_delay`，未来可支持 `development_delay`、`test_failure`、`quality_issue` 等风险来源。

### 3.5 Impact Analysis / Decision Support Module

MVP 阶段不实现完整 Decision Agent，采用规则化影响分析模块，作为 PM 的决策辅助能力。

职责：

- 输出受影响节点
- 估算简单延期影响
- 提供基础方案对比

MVP 输出：影响节点、影响范围、建议动作。

示例：

```text
输入：BGA 延期 15 天
输出：封测完成节点预计延期 15 天；客户验证阶段存在延期风险
```

不实现：

- 复杂仿真
- 成本模型
- 多资源优化
- 自动优化方案
- 成本模型
- 资源优化

### 3.6 Collaboration Module

职责：处理风险确认后的协同动作，例如飞书通知、创建任务和生成跟进事项。

MVP 先实现风险通知，Action 任务闭环按开发进度作为可选能力。

## 4. 页面与交互

### 4.1 飞书页面

飞书是主要入口：

```text
飞书聊天 → 输入异常事件 → 返回风险卡片
```

风险卡片包含：

- 项目名称
- 风险标题
- 风险等级
- 风险解释
- 触发事件
- 判断规则
- 相似案例
- 建议动作
- 推荐措施

PM 操作：

- 确认风险
- 查看影响分析
- 查看建议和协同动作

### 4.2 Web Dashboard

Dashboard 用于 Demo 辅助展示。

首页展示：

- 当前项目
- 风险数量
- 高风险事件

Risk 详情页展示：

```text
Event → Rule → Case → Risk → Recommendation
```

## 5. 数据来源与存储

### 5.1 MVP 数据来源

| 数据 | 来源 |
|---|---|
| Project | Mock |
| Milestone | Mock |
| Event | Mock / 用户输入 |
| Rule | 人工整理 |
| Case | Mock Case 库 |
| Risk | Agent 生成 |

比赛阶段不依赖真实企业系统，优先使用模拟数据和人工构造案例完成闭环。

### 5.2 数据保存位置

推荐使用 Supabase PostgreSQL：

```text
Supabase
├── projects
├── milestones
├── events
├── rules
├── cases
└── risks
```

本地 JSON 作为备用数据源和初始化数据资源。运行时业务状态以数据库为准。

## 6. 数据模型摘要

具体字段以《数据模型设计.md》为准。核心对象如下：

### Project

`project_id`、`project_name`、`stage`、`owner`、`target_date`

### Milestone

`milestone_id`、`project_id`、`name`、`status`、`delay_days`

### Event

`event_id`、`project_id`、`milestone_id`、`risk_domain`、`source_department`、`event_type`、`source`、`supplier`、`material`、`material_category`、`delay_days`、`description`、`occurred_at`、`extra_data`

Event 只描述客观事实，不包含 `risk_type` 等 AI 判断结果。

### Rule

`rule_id`、`rule_name`、`risk_domain`、`threshold`、`risk_level`、`suggest_action`

`threshold` 在 MVP 阶段以字符串保存，由后端固定逻辑或 Risk Agent 按约定解释。

### Case

`case_id`、`title`、`category`、`domain`、`context`、`problem`、`cause`、`impact`、`solution`、`result`、`risk_pattern`、`trigger_conditions`、`recommendation`、`tags`

### Risk

`risk_id`、`event_id`、`project_id`、`milestone_id`、`risk_type`、`risk_level`、`risk_title`、`risk_summary`、`risk_reason`、`matched_rules`、`matched_cases`、`case_match_reason`、`impact_analysis`、`recommendations`、`risk_status`、`review_status`、`reviewer`、`review_comment`、`created_at`

## 7. 代码与数据规范

建议目录：

```text
backend/
├── api/
├── models/
└── services/

agent/
├── risk_agent.py
└── case_retrieval.py

data/
├── mock/
└── output/

schemas/
```

规范：

- 业务逻辑分离
- Agent 输入输出严格使用 JSON
- Agent 不直接修改数据库
- 业务数据不写死在代码逻辑中
- Prompt 独立保存
- JSON 字段统一使用 `snake_case`
- ID 保证唯一
- 时间统一使用 ISO 8601

## 8. 技术栈建议

| 领域 | 建议 |
|---|---|
| 后端 | FastAPI |
| Agent 编排 | 固定 Python Workflow；LangGraph 可选 |
| LLM | OpenAI API 或主办方提供模型 |
| 数据库 | Supabase PostgreSQL |
| 前端 | 简单 HTML + JS 或 React |
| 飞书 | Bot、Webhook、Message Card |

MVP 建议使用固定流程编排，不实现复杂自治 Agent。

## 9. MVP 明确不实现

为了保证 12 天交付，不实现：

- 深度预测模型
- 复杂多 Agent 自治
- 企业权限系统
- 实时 ERP/MES 接入
- 复杂决策仿真
- 大规模向量数据库
- 完整企业数据治理

## 10. 最终 Demo 闭环

```text
PM 发送：BGA供应商延期15天
        ↓
Risk Agent 分析
        ↓
识别 High 风险
        ↓
引用 CASE001
        ↓
生成风险卡片
        ↓
飞书通知 PM
        ↓
PM 确认处理
        ↓
生成协同动作（可选）
```

验收重点：能够完整展示 `Event → Rule → Case → Risk → Feishu Card → PM 确认` 链路。
