# CarClaw

CarClaw 是一个面向车辆维修诊断场景的 AI Agent 工作流协议与产品原型。它的重点不是做通用聊天机器人，而是把诊断仪数据、DTC、维修手册证据、知识库检索和 AI 推理组织成可追溯、可复核、可增长的诊断闭环。

## 项目定位

CarClaw 关注真实维修诊断流程：

```text
DiagnosticCase -> DTC/症状 -> 证据 -> 检测步骤 -> 维修动作 -> 验证 -> 报告
```

核心目标：

- 连接诊断核心，读取 UDS/DTC 等车辆诊断信息。
- 将维修手册导入为结构化知识库，而不是停留在全文搜索。
- 基于 DTC graph、结构化 RAG、GraphRAG 和 LLM Wiki 编译层提供可追溯证据。
- 让 AI Agent 在回答时区分手册事实、诊断仪结果和 AI 推断。
- 将诊断过程沉淀为 Case 事件日志，便于审计、复盘和持续改进。

## 当前能力

- **CarClaw Agent**：支持 Kimi/Qwen 模型、工具调用、MCP 子进程、认证、会话和诊断 Case。
- **UDS MCP**：连接 diag core，读取 DTC、读取版本号、发送自定义 UDS 命令。
- **Solution MCP**：将 DTC 和维修问题转发到知识库 API，返回结构化维修证据。
- **维修知识库**：支持手册导入、章节模型、DTC 图谱、诊断树、规格/注意事项编译层。
- **Docker Compose 部署**：默认端口集中在 `6200-6204`。

## 设计原则

- **AI-friendly data**：所有关键数据尽量结构化，包含稳定 ID、版本号、source refs、confidence 和 review status。
- **Evidence first**：高风险诊断结论必须能够回到手册章节、页码、chunk、DTC graph 或工具结果。
- **Human review**：自动编译出的维修步骤、规格项和安全注意事项默认需要复核。
- **Workflow over prompt**：CarClaw 不是一个 prompt，而是一套可执行的诊断工作流协议。

## 公开内容

本仓库当前先公开：

- [`AGENTS.md`](./AGENTS.md)：CarClaw Agent 协议，描述场景边界、证据规则、工具权限和验证要求。
- [`README.md`](./README.md)：项目概要。

完整产品代码、维修手册数据、密钥和本地配置不会在公开仓库中发布。

## 注意

该项目仍在重构和产品化阶段。当前公开内容用于说明 CarClaw 的 Agent 工作协议和车辆诊断 AI 的设计方向，不代表完整可运行产品包。
