# SynapFlow 需求方案 - Data Node 提供 Sample/Dummy 数据用于 Processor Unit Test

## 1. 背景与目标
在当前数据流画布中，Processor 节点脚本调试高度依赖上游 Data Node 的真实数据。该方式会带来三类问题：
- 调试慢：真实数据量大，反馈周期长。
- 风险高：调试阶段可能误触生产数据或产生额外成本。
- 重现难：缺乏稳定、可复用的测试输入样本。

本需求目标是让 Data Node 能管理可控的 Sample/Dummy 数据，并在 Processor 节点中一键用于 Unit Test，实现快速、安全、可重复的测试闭环。

## 2. 适用用户与核心场景
- 数据分析师：验证 Join、过滤、聚合逻辑是否正确。
- 数据工程师：在上线前对 Processor 脚本做快速回归。
- AI 协作开发者：反复让 Agent 改写脚本时，使用稳定样本做对比验证。

核心场景：用户在 Processor 节点修改脚本后，点击运行单测，系统自动使用上游 Data Node 的 Test Data 执行并返回测试结果，而非读取真实全量数据。

## 3. 用户旅程 (Journey)
1. 用户创建或选中一个 Data Node。
2. 用户在 Data Node 侧边面板进入 Test Data 标签页。
3. 用户选择一种方式准备测试数据：
   - AI 按 Schema 自动生成。
   - 手动粘贴 JSON/CSV。
   - 从真实数据抽样并脱敏。
4. 用户保存 Test Data，并可命名版本（如 v1-smoke、v2-join-case）。
5. 用户进入 Processor 节点，编写或由 Agent 生成脚本。
6. 用户点击 Run Unit Test。
7. 系统自动解析上游依赖 Data Node，注入对应 Test Data 运行脚本。
8. Processor 返回冒烟测试结果：
  - 运行是否成功（无异常）
  - 是否超时
  - 基础执行日志
9. 若冒烟通过，系统生成/刷新输出 Data Node（包含本次 Unit Test 输出数据与执行元信息）。
10. 用户点击进入输出 Data Node，查看 Unit Test 输出。
11. 用户手动或通过 Agent 辅助标注“是否符合预期”，或录入期望输出。
12. 系统在输出 Data Node 中执行断言并给出结果，用户据此决定是否回到 Processor 继续调整。

## 4. 交互设计
### 4.1 Data Node 面板
- 新增标签：Test Data。
- 区块 A：数据来源方式选择（AI 生成 / 手动输入 / 抽样脱敏）。
- 区块 B：数据预览（分页表格 + 行数统计 + 字段类型校验提示）。
- 区块 C：版本管理（新建版本、复制版本、删除版本、设为默认测试版本）。
- 区块 D：质量提示（空值占比、类型冲突、主键重复等轻量检测）。

### 4.2 Processor Node 面板
- 在脚本编辑区主操作区新增按钮：Run Unit Test。
- 增加模式开关：
  - Test 模式：读取上游默认测试版本。
  - Prod 模式：读取真实数据（保留原有执行能力）。
- Processor 仅展示冒烟测试状态：
  - Smoke Status：Pass/Fail（失败原因仅限异常或超时）。
  - Logs：执行日志与错误堆栈。
  - Output Link：跳转到本次运行对应的输出 Data Node。

### 4.3 输出 Data Node 面板（断言工作区）
- 新增 Test Result 标签：展示本次 Unit Test 的输出数据、运行参数与时间戳。
- 新增 Expectation 标签：
  - 手动标注预期（字段值、行数范围、关键记录）。
  - 通过 Agent 生成或补全期望输出。
- 新增 Assert 操作：在输出 Data Node 内触发断言执行并返回通过/失败明细。
- 断言结果与期望版本绑定，支持复跑和历史对比。

### 4.4 可用性与无障碍
- 所有主要操作提供键盘可达性（Tab 顺序清晰，Enter 可触发主按钮）。
- 失败状态使用文字 + 图标双重提示，不只依赖颜色。
- 表格预览支持列名朗读友好标签，便于辅助读屏。

## 5. 功能需求 (Functional Requirements)
1. Data Node 必须支持存储至少一个 Test Data 版本。
2. 系统必须支持基于 Schema 自动生成 Dummy 数据。
3. Processor 在 Test 模式下执行时必须优先读取上游 Test Data。
4. 若上游存在多个 Data Node，系统必须对每个输入源做独立映射。
5. Processor 的 Unit Test 判定只包含冒烟维度：无异常、未超时。
6. Unit Test 执行后必须生成结构化输出并挂载到输出 Data Node。
7. 断言必须在输出 Data Node 内执行，支持手动标注或 Agent 辅助定义期望输出。
8. 用户必须可重复运行相同测试并得到可追踪历史记录。

## 6. 非功能需求 (Non-Functional)
1. 单测反馈时长目标：中小样本下 3 秒内返回首屏结果。
2. 测试数据与生产数据逻辑隔离，避免误用生产路径。
3. 支持最小化权限原则：仅有编辑权限的用户可修改 Test Data。

## 7. 验收标准 (Acceptance Criteria)
1. 给定一个含 Schema 的 Data Node，用户可在 30 秒内生成并保存可用 Dummy 数据。
2. 给定连接了 2 个上游 Data Node 的 Processor，点击 Run Unit Test 后，系统能正确读取两个输入的默认测试版本并完成执行。
3. 当 Processor 报错或超时时，Processor 面板必须仅展示冒烟失败信息（异常/超时）及日志，不出现断言模块。
4. 当冒烟通过后，用户进入输出 Data Node 可完成期望标注并执行断言，失败时可查看失败断言名称、期望值、实际值。
5. 用户连续修改脚本并重跑 3 次后，测试历史可回看每次输入版本、输出摘要与断言结论。

## 8. 风险与待确认
- 是否允许 Test Data 与真实 Schema 暂时不一致（宽松模式）？
- 输出 Data Node 中断言能力首版范围：仅支持行数/空值/字段存在，还是开放表达式断言？
- 版本数量上限与存储成本控制策略。

## 9. 建议迭代拆分
1. M1：单 Data Node 测试数据 + Processor 冒烟单测（异常/超时）+ 输出 Data Node 预览。
2. M2：输出 Data Node 断言工作区（手动标注 + 基础断言）+ 测试历史。
3. M3：Agent 辅助期望输出 + 高阶断言表达式 + 团队共享测试集。