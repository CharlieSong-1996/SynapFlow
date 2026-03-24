# 架构设计方案与框架比选 - SynapFlow MVP

## 1. 需求分析与核心架构挑战
根据前期 PM 的产品文档（Idea 1-4），SynapFlow 是一个由 AI 深度赋能的画布式数据流处理工具核心能力分为两部分：前端负责高度可交互的无代码/低代码画布边界展示（数据面板、Agent 对话框、SVG/HTML 子集的可视化渲染），后端负责 AI Agent 上下文管理与代码生成、安全的数据处理脚本沙箱体系（Python/SQL）。

**核心技术挑战：**
* **前端 (Frontend)**: 复杂的画布状态同步、多节点拓扑解析、节点间的高频通信体验（包括对 Stale 节点样式的差异化渲染、Agent Chatbox 混合界面）。
* **后端 (Backend)**: 在受限环境中安全且高效地执行用户或 AI 动态生成的不可信脚本语言（Python/SQL）；调度与管理不同的 Processor 节点执行生命周期（Strict Mapping, Addition Only 等增量与全量计算机制）。

## 2. 前端架构比选 (Frontend Flow Builder & UI)

前端需要一个强大的节点流图引擎以及灵活的组件体系，并且能够很好地处理多状态的 Canvas 逻辑。

| 评估维度 | React Flow (基于 React) | Vue Flow (基于 Vue) | X6 / G6 (AntV) |
|---|---|---|---|
| **学习成本** | 如果团队熟悉 React，则极低；其 Hooks API 友好且直观。 | 如果熟悉 Vue，较易上手；与 React Flow 完全类似。 | 较高；具备复杂的原生 Canvas/SVG 概念。 |
| **社区支持** | 活跃度极高，大量现有低代码/AI产品默认首选 (如 LangFlow, Dify)。 | 活跃度尚可，生态略逊于 React Flow。 | 充足，尤其在国内大厂中有着广泛成熟应用。 |
| **性能表现** | 优秀，支持数百个节点的虚拟滚动与局部渲染优化。 | 优秀，Vue 3 的响应式系统带来极低开销。 | 极强，底层优化适配大规模图挖掘和拓扑渲染。 |
| **与项目集成度** | 完美契合 "节点卡片嵌对话框/数据表" 这样的重自定义 UI 需求（基于 HTML 节点）。 | 好，基于 Vue 组件注入自定义卡片节点体验顺畅。 | 尚可，但在自由嵌入复杂业务级 React/Vue 组件时略显间接。 |
| **长期维护性** | 极佳，背靠庞大 React 社区及官方长期全职维护支持。 | 优良，跟随 Vue 生态同步更迭。 | 优良，有企业级团队维护。 |

**🏆 前端推荐结论：React + React Flow + Zustand**
* **理由**：你的需求中涉及【内嵌 Agent 对话框】、【可视化 SVG/HTML 引擎直接渲染】以及【测试数据表格预览】。React Flow 允许完全基于自定义 React Component 构筑 DOM 节点而非传统的 Canvas 绘制。结合 Zustand 状态管理，可以极大地简化不同层级状态（Data Node 的 Meta，Processor 的连线及运行状态，执行拓扑图等）的分发。

## 3. 后端架构比选 (Backend API & Execution Engine)

后端系统既要处理来自前端的 HTTP/WebSocket 通信，还要作为大模型对话管理（LLM Orchestration）和动态数据执行引擎（Local/Sandbox）的统筹层。

| 评估维度 | Python (FastAPI + LangChain/LlamaIndex) | Node.js (NestJS / Express + LangChain.js) | Go (Fiber/Gin + 独立沙箱) |
|---|---|---|---|
| **学习成本** | 极低（AI 研发团队标配），数据分析、Python 脚本生成与执行浑然一体。 | 中等（若团队背景多为前端/JS），依赖 TypeScript 及 JS 生态。 | 较高（涉及强类型、并发模型等，且大模型 SDK 原生支持少）。 |
| **社区支持** | 数据科学和 LLM/Agent 生态（LangChain, Autogen 等）的绝对核心阵地。 | 全栈友好，生态极其繁荣，但在 Python 原生数据处理（如 Pandas）生态上较为被动。 | 适合做高并发的 API Gateway，AI 生态需大量造轮子。 |
| **性能表现** | 处理 HTTP 较快 (ASGI)，但在重计算时受限 (GIL)；需要依赖任务队列和多进程。 | V8 引擎异步处理请求能力极佳，但在处理重型数据转换时仍需调起外部 Python 进程。 | 极致的高并发响应和原生协程支持性能极佳。 |
| **与项目集成度** | **完美**。目前 Agent 需要生成并执行 Python/SQL 及处理 DataFrame(如 Pandas/DuckDB)。后端本身就是 Python 环境可带来天然的执行便利度。 | 一般。每次动态执行 Python 脚本都需要繁重的进程间隔离/通信机制。 | 较差。主要定位为代理层，仍需要另外编写 Python/SQL 执行 worker。 |
| **长期维护性** | 优秀，架构随 AI 和数据计算生态的演进最为自然。 | 中等，架构上必须拆分为 Node API + Python Worker 会增加部署和维护复杂性。 | 对于初期阶段可能导致过度工程，架构复杂度高。 |

**🏆 后端推荐结论：Python + FastAPI + DuckDB (本地内存型测试执行) + Celery/RQ**
* **理由**：SynapFlow 本质是“数据分析”与“AI 代码生成”。因为底层产出最多的是 Python 脚本或 SQL，因此使用基于 Python 的后端是最直观的决策。
    * **FastAPI** 负责承接画布的拓扑 API 及 Chatbot 实时对话 (WebSocket)。
    * 可以采用轻量级的 **DuckDB** 充当针对 Data Node Test Data 单元测试（Idea 2）的高效内存级别 SQL/数据验证引擎。
    * 由于含有“单测运行（Run Unit Test）”或基于增量/重算的执行引擎（Idea 4），使用 Python 后台能最便捷地搭建一个轻量级沙箱或子进程处理模型（Subprocess/Docker）来执行这些 Processor Nodes 中的动态脚本。

## 4. 整体架构拓扑图

*   **UI Layer (React Flow / Next.js)**: 管理画布节点，承接数据节点表格、Agent Chat 的展示。
*   **State Management (Zustand)**: 同步本地画布的 DAG（有向无环图）的拓扑关系配置。
*   **API Gateway (FastAPI)**: RESTful 用于拉取项目、保存节点 Meta 和 Script，WebSocket 用于连通前端界面的 Agent 交互回复。
*   **AI Orchestration (LangChain / LlamaIndex)**: 给每个 Processor Node 分配和维护记忆实例，执行智能分析与代码生成。
*   **Execution Runtime (Python Subprocess / Docker Sandbox + DuckDB)**: 根据前台传来的“冒烟测试（Test Data）”执行对应脚本返回状态响应（Pass/Fail/Logs）。