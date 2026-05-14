# CarClaw 社区协议

CarClaw Community Protocol 是一套面向 **车辆维修诊断 AI Agent** 的开源工作流协议。

它不是一个通用聊天提示词，而是把车辆症状、DTC 故障码、维修手册证据、诊断推理和维修报告组织成一个可追溯、可复核的诊断闭环。

```text
车辆症状 / DTC
  -> 证据检索
  -> 诊断推理
  -> 下一步检查
  -> 维修记录
  -> 可审计报告
```

## 文档入口

| 文档 | 中文 | English |
|---|---|---|
| 项目说明 | [README.md](./README.md) | [README.en.md](./README.en.md) |
| Agent 协议 | [AGENTS.md](./AGENTS.md) | [AGENTS.en.md](./AGENTS.en.md) |
| 诊断 Skill | [SKILL.md](./SKILL.md) | [SKILL.en.md](./SKILL.en.md) |

## 产品截图

### Agent 工具面板

![CarClaw Agent 工具面板](./assets/carclaw-agent-tools.png)

### 读取 DTC 与工具调用

![CarClaw Agent 读取 DTC](./assets/carclaw-agent-dtc-read.png)

### 证据优先诊断报告

![CarClaw Agent 诊断报告](./assets/carclaw-agent-report.png)

### DTC 复核工作台

![CarClaw DTC 复核工作台](./assets/carclaw-dtc-review.png)

### 诊断 Case 工作台

![CarClaw 诊断 Case 工作台](./assets/carclaw-diagnostic-case.png)

## 这个仓库包含什么

- `AGENTS.md`：车辆诊断 Agent 的公开操作协议，定义证据规则、安全边界、工具契约和输出要求。
- `SKILL.md`：只读版车辆诊断 Skill，可适配到 MCP、function calling、本地脚本或其他 agent runtime。
- `README.md`：项目定位、开源边界和使用方式。

公开版刻意保持轻量，适合作为汽车诊断 Agent 的协议样板、教学材料、安全评审材料或 demo 起点。

## 这个仓库不包含什么

本公开仓库不包含：

- 原厂或第三方专有维修手册
- 客户维修数据
- 车型专用 ECU 配置
- CAN ID / 响应 ID 数据库
- 真实硬件适配器
- 商业解析规则
- 生产部署代码
- 密钥或私有环境配置

这些内容应保留在私有部署、客户项目或商业版本中。

## 社区版适用范围

社区版适合：

- mock 诊断环境
- 只读 DTC 工作流
- 维修手册证据检索
- Agent 指令设计
- 安全边界评审
- 概念验证 demo

社区版不适合直接控制真实车辆。真实车辆诊断需要额外的安全审查、明确用户授权、硬件验证和合规审查。

## 推荐架构

```text
用户 / 技师
  -> 诊断 Agent
  -> 诊断 Skill
  -> 只读诊断工具
  -> 知识库检索工具
  -> 证据优先报告
```

推荐工具能力：

- `diagnostic_connect`：连接 mock 或已授权的诊断数据源。
- `diagnostic_read_dtc`：读取 DTC 故障码。
- `knowledge_query_dtc_solution`：查询单个 DTC 的证据和维修线索。
- `knowledge_search`：按症状、部件或系统检索维修手册证据。
- `case_record_event`：可选，记录诊断 Case 的审计事件。

这些逻辑工具可以映射到 MCP tools、function tools、HTTP API、本地脚本或你自己的 agent runtime。

## 安全原则

- 维修手册、结构化证据和已验证诊断数据是事实源。
- 明确区分：用户描述、工具结果、手册证据、AI 推断。
- 不编造 DTC 含义、扭矩、油液规格、针脚、电压范围或维修步骤。
- 知识库证据不足时，直接说明“当前知识库证据不足”。
- 社区版默认不包含写操作。
- 不在社区 Skill 中授权清码、写 ECU 配置、动作测试、标定、刷写或自定义 UDS 命令。

## 开源策略

这个仓库是公开协议层。

建议的 open core 边界：

- 开源：协议、只读 Skill、mock 工具、demo 数据、报告结构。
- 商业或私有：客户手册、数据清洗、品牌/车型解析规则、真实硬件适配、团队流程、部署、维护和支持。

## 状态

当前是早期公开协议包。它可以作为 Agent 工作流设计参考，但不是完整车辆诊断产品。
