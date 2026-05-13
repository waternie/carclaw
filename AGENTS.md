# CarClaw Agent Protocol

CarClaw 是面向汽车维修诊断的垂直 AI 工作台，不是通用聊天站，也不是普通文档搜索站。任何 AI-agent 在本仓库工作时，都要围绕真实诊断闭环维护产品一致性：`DiagnosticCase -> DTC/症状 -> 证据 -> 检测步骤 -> 维修动作 -> 验证 -> 报告`。

## 垂直场景定位

- 核心用户：维修技师、维修资料整理者、诊断工具开发者、小型维修团队。
- 核心任务：把车辆故障现象、诊断仪数据、维修手册证据和 AI 推理组织成可审计的诊断 Case。
- 核心壁垒：DTC 证据图、维修手册 source 追溯、UDS 诊断工具链、Case 事件日志、可复用诊断 skill。
- 不做泛化：不要把 CarClaw 改成通用知识库、通用 Agent 平台、通用客服机器人或纯 prompt 工具。

## 入口顺序

1. 先读 `CARCLAW_ARCHITECTURE.md`，理解整体组件、端口、MCP 工具和诊断流程。
2. 产品化或业务对象相关任务，读 `docs/carclaw-product-refactor-design.md` 和 `docs/kb-refactor-progress.md`。
3. 知识库、RAG、GraphRAG、Wiki 编译相关任务，读 `docs/adr-kb-rag-graphrag-llm-wiki.md`、`docs/kb-redesign-blueprint.md` 和 `docs/manual-import-pipeline.md`。
4. Agent 对话、工具调用、权限、Case 事件相关任务，读 `ai-agent/README.md`、`ai-agent/MCP_SETUP.md` 和 `ai-agent/agent.py`。
5. MCP 工具相关任务，读 `uds-mcp/README.md` 或 `solution-mcp/README.md`。
6. 诊断 skill 相关任务，读 `.cursor/skills/vehicle-diagnosis/SKILL.md` 和 `.cursor/skills/vehicle-diagnosis/examples.md`。

## 目录职责

- `ai-agent/`: CarClaw Agent 后端、对话 UI、认证、会话、Case 事件、模型调用和 MCP 子进程管理。
- `.cursor/skills/`: 诊断运行时 skill。这里定义用户意图、触发词、工具顺序和报告格式。
- `uds-mcp/`: 诊断通信 MCP。负责连接 diag core、读取 DTC、读取版本、自定义 UDS 命令。
- `solution-mcp/`: 维修方案 MCP。负责把 DTC/关键词/章节查询转发到知识库 v2 API。
- `kb_wiki_llm/`: 维修知识库平台。包括 Vue 前端、Node 后端、PDF 手册导入、v2 corpus/index/model/DTC graph/wiki pipeline。
- `docs/`: 产品设计、ADR、知识库重构蓝图和导入流程。架构争议优先沉淀到这里。
- `docker/` 和 `docker-compose.yml`: 本地/产品化部署入口，默认端口范围是 `6200-6204`。
- `Rule/`: 语言和前端设计规则。做对应技术栈修改时要参考。

## Source of Truth

- 维修事实的第一事实源是原始手册、结构化 corpus、chunk、entity、relation、evidence span 和 DTC graph。
- LLM Wiki 是编译后的阅读层，不是事实源。
- AI 回答可以总结，但不能凭空制造 DTC 含义、扭矩、油液、针脚、电压、维修步骤或安全警告。
- 没有证据时，明确说“当前知识库证据不足”，不要输出看似确定的维修结论。
- 每个高风险结论都应能回到章节、页码、chunk、证据片段或诊断工具结果。

## 诊断工作流协议

当用户请求车辆诊断、故障灯、DTC、维修建议或全车检查时，优先遵循：

1. 建立或绑定 `DiagnosticCase`。
2. 记录用户症状、车辆信息和上下文。
3. 如需读车，先连接 diag core，再读取 DTC；不要跳过连接步骤。
4. 对每个 DTC 查询 `query_dtc_solution`。
5. 用知识库证据、诊断仪结果和 AI 推断分别组织结论。
6. 输出下一步检测，而不是直接承诺故障原因唯一确定。
7. 关键行为写入 Case 事件：用户消息、工具调用、工具结果、知识库查询、AI 建议、人工复核、维修动作。

## 工具和权限边界

- `uds_connect`、`uds_read_dtc`、`uds_read_version` 属于读诊断信息工具，默认可用于 mock diag core 或明确授权的诊断连接。
- `uds_send_custom` 可能影响车辆状态，必须有明确用户授权、目标 ECU、CAN ID、服务命令和风险说明后再使用。
- 清除 DTC、写入配置、执行动作测试、刷写、标定、复位学习等高风险动作，必须要求用户明确确认；当前没有明确工具时不要模拟执行。
- 默认开发和演示使用 `mock-diag-core`。不要假设本地连接的是真车。
- MCP 配置必须保持可迁移，优先使用相对路径，不写入本机绝对路径。
- 不要把真实 API key、诊断密钥、数据库密码写入仓库；使用 `.env` 或 Docker 环境变量。

## 输出合同

诊断类回答应尽量包含：

- 诊断概览：车辆/Case、症状、已读 DTC、严重程度。
- 已知事实：来自用户、诊断仪、维修手册或知识库的事实分别列出。
- 故障详情：每个 DTC 的描述、证据、可能原因、检测步骤、维修动作。
- 不确定性：缺少哪些车辆信息、数据流、测量值或手册证据。
- 下一步：建议先做的 1-3 个检查动作，按安全性和证据强度排序。
- 来源：章节、页码、chunk、DTC graph、工具结果或 Case 事件。

避免：

- 把 AI 推断写成手册事实。
- 给没有证据的数值型建议。
- 在未读取 DTC 或未查询知识库时输出最终诊断。
- 用“应该就是”“必然是”这类不可审计表述。

## 知识库导入与编译

- 新手册必须通过 source registry 注册，不要直接手改生成后的 normalized 数据。
- 运行导入和重建时，从 `kb_wiki_llm/backend` 执行：

```powershell
npm run kb:manuals -- scan
npm run kb:manuals -- register --file ../../book/example.pdf --id example --title "Example Manual"
npm run kb:manuals -- import --id example
npm run kb:rebuild:v2
```

- DTC 是最高优先级实体。任何知识库改动都不能降低 DTC 查询、证据引用和复核状态的可靠性。
- 旧 demo 数据、硬编码车型、硬编码手册名、不可追溯摘要和无来源规格项可以删除或迁移。
- 高风险页面和规格项默认需要 `reviewed: false` 或人工复核标记。

## 编码规则

- 保持现有技术栈：FastAPI/Python、Node/Express、Vue 3/Element Plus、TypeScript MCP、Docker Compose。
- 后端改动要保留认证、Case 归属和事件审计。
- 前端要从“聊天页”逐步走向诊断工作区：Case、车辆、DTC、证据、工具调用结果和下一步动作都应可见。
- 新 API 优先走 `/api/v2/...`，旧 API 只做兼容转发，不再扩展旧数据模型。
- 结构化数据优先使用稳定 ID、版本号、source refs、confidence 和 review status。
- 不要让模型输出直接覆盖 source of truth；模型输出进入草稿、复核或编译层。

## 常用命令

根目录：

```powershell
.\start-carclaw.bat
.\stop-carclaw.bat
docker compose up -d --build
docker compose down
```

Agent：

```powershell
cd ai-agent
python start.py
python test_mcp.py
python test_vehicle.py
python -m py_compile agent.py auth.py
```

知识库：

```powershell
cd kb_wiki_llm
npm run install:all
npm run dev
npm run build
cd backend
npm test
npm run kb:rebuild:v2
```

Solution MCP：

```powershell
cd solution-mcp
npm install
npm run build
npm run start
```

UDS MCP：

```powershell
cd uds-mcp
pip install -r requirements.txt
python -u server.py
```

## 验证要求

- 修改 Agent 后，至少运行 Python 编译检查；涉及工具调用时运行 `ai-agent/test_mcp.py` 或对应最小复现。
- 修改 `solution-mcp` 后，运行 `npm run build`。
- 修改知识库后端后，运行 `npm test`，并在需要时运行 `npm run kb:rebuild:v2`。
- 修改前端后，运行 `npm run build`。
- 修改 Docker 或端口配置后，验证 `docker compose up -d --build` 能启动相关服务。

## 将 CarClaw 封装成工作流协议包

CarClaw 的协议包由以下部分组成：

- `AGENTS.md`: 项目级工作协议，定义场景、边界、证据规则、工具权限和验证命令。
- `.cursor/skills/vehicle-diagnosis/SKILL.md`: 运行时诊断 skill，定义触发词、工具顺序和报告模板。
- `ai-agent/mcp-config.json`: 工具协议入口，连接 UDS MCP 和 Solution MCP。
- `docs/adr-kb-rag-graphrag-llm-wiki.md`: 知识源、RAG、GraphRAG 和 Wiki 编译的架构约束。
- `kb_wiki_llm/backend/pipeline/`: source 注册、导入、索引、DTC graph 和 wiki 编译流水线。
- `DiagnosticCase` 与事件日志：真实诊断过程的状态、审计和复盘层。

这个封装的目标不是“卖一个 prompt”，而是让任意 agent 进入仓库后都能按同一套汽车诊断流程工作，并持续沉淀可复用的诊断资产。
