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
- [2026-03-23] 新增需求方向：Data Node 需提供 Sample/Dummy 数据，供 Processor 在 Test 模式执行 Unit Test。
- [2026-03-24] Unit Test 断言后置：Processor 仅负责冒烟判定（无异常、未超时）；断言在输出 Data Node 中由用户手动或 Agent 辅助定义期望后执行。
- [2026-03-24] 数据节点细分确认：自由数据节点、输出型数据节点；可视化采用方案1（作为数据节点输出形态），由框架强制约束渲染格式（SVG/HTML 子集）并由画布直接渲染。
- [2026-03-24] 过时输出节点展示规则确认：默认折叠并以虚线关联；若存在下游 Processor 依赖则不折叠，仅连线虚线化。
- [2026-03-24] 处理器执行类型确认：Strict Mapping / Addition Only / Recreation Only / Self Managed 四分类。
- [2026-03-24] 测试能力新增：Addition Only 开放幂等性测试；Self Managed 开放输入无序测试与增删等价测试。
- [2026-03-24] Processor 处理程序变更但 Meta Data 不变时，用户必须选择历史数据处理策略（仅新数据 / 全量重算 / 保留版本分界）。
- [2026-03-24] Self Managed 若支持历史版本管理，必须提供 Manage History API。
- [2026-03-24] Processor 的 Meta Data 编辑与处理程序编辑职责分离：Chat Agent 不直接编辑 Meta Data，Meta Data 编辑需在专属对话中完成。

## 4. 需求规划与 PRD 索引 (Requirements & PRDs)
*(在此记录全局需求，并关联 `planning/product_manager/idea-{id}.md` 文件)*
- 已归档：基础画布与节点拖拽 MVP 体验 -> [planning/product_manager/idea-1.md](../planning/product_manager/idea-1.md)
- 已归档：Data Node 提供 Sample/Dummy 数据供 Processor Unit Test -> [planning/product_manager/idea-2.md](../planning/product_manager/idea-2.md)
- 已归档：数据节点分类、过时输出展示与可视化节点方案1 -> [planning/product_manager/idea-3.md](../planning/product_manager/idea-3.md)
- 已归档：处理器执行类型、事件调度与专项测试矩阵 -> [planning/product_manager/idea-4.md](../planning/product_manager/idea-4.md)

## 5. 待办事项与发散想法 (Backlog & Ideas)
- 明确目标用户画像。
- 梳理首个开箱即用的“Aha Moment”主线互动流程。
