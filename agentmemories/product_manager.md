# 产品经理全局记忆库

## 1. 项目概览 (Project Context)
- **当前项目**: 类似 n8n 画布的数据处理工具 (AI 驱动模型)
- **核心理念**: 将节点极致简化为两大类——**数据节点**（Data Nodes）和**处理器节点**（Processor Nodes）。旨在降低用户认知负荷，提供清晰的数据流传导体验。
  - **数据节点 (Data Node)**: 代表一个 SQL 表或 Query View，附带数据的 Meta 信息（如列描述、Schema 等）。作为数据流的起点和终点。
  - **处理器节点 (Processor Node)**: 本质是一段脚本程序（以及背后的 Agent 对话上下文和记忆）。接受一个或多个数据节点作为输入，通过脚本执行产生新的输出数据节点。支持通过拖拽连线定义输入，支持通过内置 Agent 对话直接编写和修改脚本。

## 2. 核心交互模式 (User Interaction Patterns)
*(待补充：记录用户偏好的交互设计模式、UI/UX 规范等)*

## 3. 关键决策记录 (Key Decisions)
- [2026-03-23] 确认核心节点双轨制：数据节点负责输入/输出，处理器节点负责逻辑转换与计算。

## 4. 需求规划与 PRD 索引 (Requirements & PRDs)
*(在此记录全局需求，并关联 `planning/product_manager/idea-{id}.md` 文件)*
- 当前讨论中：基础画布与节点拖拽 MVP 体验。

## 5. 待办事项与发散想法 (Backlog & Ideas)
- 明确目标用户画像。
- 梳理首个开箱即用的“Aha Moment”主线互动流程。
