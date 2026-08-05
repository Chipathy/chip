# PM Agent 全流程风险副驾驶 MVP PRD v1.0

## 1. 项目概述

### 1.1 项目名称

PM Agent 全流程风险副驾驶（PM Risk Copilot）

### 1.2 项目背景

PM 并非只负责供应链协调，而是整个芯片项目生命周期中的横向协调角色，需要连接研发、测试、供应链、质量、生产及其他上下游角色。项目风险来源包括：

- 供应链延期
- 研发延期
- 测试异常
- 质量问题
- 项目节点偏移
- 资源协调问题

当前项目风险管理通常存在以下问题：

- 信息分散在不同沟通渠道
- 风险发现滞后
- 历史经验无法快速复用
- PM 协调成本高

本项目通过 AI 副驾驶帮助 PM 从“被动获取信息”转变为“主动识别风险并推动处理”。

## 2. 产品目标与定位

### 2.1 MVP 目标

构建一个面向芯片项目 PM 的全流程风险副驾驶，完成以下闭环：

```text
异常事件输入
    ↓
Risk Agent 分析
    ↓
历史案例匹配
    ↓
生成风险卡片
    ↓
PM 确认
    ↓
协同处理
```

### 2.2 产品定位

产品定位为面向芯片项目 PM 的全流程风险副驾驶，不替代 PM 决策，而是提供：

- 风险判断
- 历史经验
- 处理建议
- 信息整理和协同支持

供应链风险仅作为 MVP 阶段的首个落地场景，未来可扩展研发、测试、质量和生产风险。

## 3. 用户角色

### 3.1 PM（项目经理）

主要用户，关注：

- 查看项目风险
- 判断风险等级
- 获取处理建议
- 推动协同动作

### 3.2 供应链负责人

关注：

- 了解供应异常
- 查看项目影响
- 跟进风险解决进度

## 4. 核心用户场景

### 4.1 MVP 场景：BGA 供应商延期导致封测节点风险

PM 在飞书发送：

```text
BGA供应商延期15天
```

系统处理：

1. 创建 Event
2. 匹配 Rule
3. 检索历史 Case
4. Risk Agent 分析
5. 生成 Risk
6. 返回飞书风险卡片

风险卡片示例：

```text
风险：BGA供应延期风险
等级：High
原因：延期超过14天，触发规则 R001
历史案例：CASE001
建议：启动第二供应商验证
```

选择该场景是因为风险事件结构化程度高、存在明确规则、有历史案例支持，并且能够完整展示 Risk Agent 闭环。

## 5. 功能模块

### 5.1 Risk Agent（全流程风险分析 Agent，核心）

职责：理解项目风险事件，不限定风险领域。MVP 阶段优先实现 `supplier_delay`。

输入：

- Event
- Rule
- Case
- Project
- Milestone

支持的风险来源示例：

| 领域 | `event_type` 示例 |
|---|---|
| 供应链 | `supplier_delay` |
| 研发 | `development_delay` |
| 测试 | `test_failure` |
| 质量 | `quality_issue` |

输出：Risk。

核心能力：

1. 风险识别：判断是否存在风险及风险等级。
2. 案例辅助分析：通过 RAG 查找相似历史案例。
3. 风险解释：输出风险原因、影响和处理建议。

### 5.2 Collaboration Agent（协同）

输入：Risk。

输出：

- 飞书通知
- 跟进任务
- 会议建议

MVP 优先实现风险通知，任务创建可按进度实现。

### 5.3 Impact Analysis / Decision Support Module

这是决策辅助模块，不是完整 Decision Agent。

功能：

- 输出受影响项目节点
- 分析简单延期影响
- 提供基础处理建议

MVP 输出：影响节点、影响范围、建议动作。

不实现：

- 复杂排期模拟
- 成本优化
- 多方案预测
- 自动优化方案
- 成本模型
- 资源优化

## 6. 用户交互设计

### 6.1 飞书入口

飞书是主要入口：

```text
用户消息 → 飞书 Bot → Risk Agent → 返回风险卡片
```

### 6.2 风险卡片

展示内容：

- 项目
- 风险标题
- 风险等级
- 风险分析
- 触发事件
- 匹配规则
- 历史案例
- 建议动作
- 推荐处理方式

PM 操作：

- 确认风险
- 标记风险状态
- 查看影响分析

## 7. 数据来源与数据模型

### 7.1 MVP 数据来源

| 数据 | 来源 |
|---|---|
| Project | Mock |
| Milestone | Mock |
| Event | Mock / 用户输入 |
| Rule | 人工整理 |
| Case | Case 库 |
| Risk | Agent 生成 |

### 7.2 数据模型

具体定义以《数据模型设计.md》为准。

#### Project

`project_id`、`project_name`、`stage`、`owner`、`target_date`

#### Milestone

`milestone_id`、`project_id`、`name`、`status`、`delay_days`

#### Event

`event_id`、`project_id`、`milestone_id`、`event_type`、`supplier`、`material`、`delay_days`、`description`、`occurred_at`、`extra_data`

Event 只描述客观事实，不包含 AI 风险分类结果。

#### Rule

`rule_id`、`rule_name`、`threshold`、`risk_level`、`suggest_action`

#### Case

`case_id`、`title`、`context`、`problem`、`cause`、`impact`、`solution`、`result`、`risk_pattern`、`trigger_conditions`、`recommendation`、`tags`

#### Risk

`risk_id`、`event_id`、`project_id`、`milestone_id`、`risk_level`、`risk_title`、`risk_summary`、`risk_reason`、`matched_rules`、`matched_cases`、`case_match_reason`、`impact_analysis`、`recommendations`、`risk_status`、`review_status`、`reviewer`、`review_comment`、`created_at`

## 8. MVP 范围与优先级

### P0：必须完成

- Risk Agent
- Case RAG
- Risk JSON 生成
- 飞书风险卡片
- PM 审核流程

### P1：建议完成

- 协同通知
- 任务创建
- 简单影响分析

### P2：暂不实现

- 自动决策模拟
- ERP/MES 实时接入
- 复杂预测模型
- 大规模知识库

## 9. 技术需求

### 9.1 后端

推荐 FastAPI，负责：

- API
- Agent 调用
- 数据处理

### 9.2 数据库

推荐 Supabase PostgreSQL，存储：

- Project
- Milestone
- Event
- Rule
- Case
- Risk

本地 JSON 用于 Mock 和初始化数据。

### 9.3 AI 能力

LLM 负责：

- 风险分析
- 案例总结
- 建议生成

### 9.4 飞书

用于：

- 用户入口
- 消息卡片
- 协同通知

## 10. 核心 Demo 流程

```text
PM 输入：BGA供应商延期15天
        ↓
系统生成 Event
        ↓
Risk Agent 分析
        ↓
匹配 R001
        ↓
召回 CASE001
        ↓
生成 High 风险
        ↓
返回飞书卡片
        ↓
PM 确认
        ↓
执行协同动作
```

## 11. 成功标准

Demo 成功标准：

- 用户可以从飞书输入异常事件。
- 系统能够识别风险并输出风险等级。
- 系统能够引用相关历史案例。
- 系统能够生成结构化 Risk JSON。
- 飞书能够展示风险卡片。
- PM 可以确认风险并推进后续处理。
