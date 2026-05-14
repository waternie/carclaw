---
name: vehicle-diagnosis-community
description: 只读车辆诊断工作流。适用于用户询问车辆症状、故障灯、DTC、诊断检查或维修建议。本社区 Skill 会检索 DTC 和维修手册证据，并生成证据优先的诊断报告。
---

# 车辆诊断社区 Skill

这是 CarClaw 风格 Agent 的公开只读诊断 Skill。

它适用于 mock 环境、已授权的只读诊断数据和维修手册证据检索。它不授权清码、写 ECU 配置、动作测试、标定、刷写或自定义 UDS 命令。

## 触发示例

当用户这样提问时使用本 Skill：

- “帮我诊断一下车辆故障”
- “发动机故障灯亮了”
- “读一下故障码”
- “P0300 是什么意思？”
- “根据这个 DTC 给我诊断建议”
- “帮我查一下这个症状对应的维修手册证据”
- “Check this vehicle fault”
- “The engine light is on”
- “Read the fault codes”

## 所需工具能力

可以把这些逻辑工具映射到你的 MCP server、function tools、本地 API 或脚本。

### `diagnostic_connect`

连接 mock 或已授权的诊断数据源。

建议输入：

```json
{
  "target": "mock-or-authorized-source"
}
```

期望输出：

```json
{
  "connected": true,
  "source": "mock",
  "metadata": {}
}
```

### `diagnostic_read_dtc`

从已授权的数据源读取 DTC。

建议输入：

```json
{
  "ecu": "optional",
  "profile": "optional"
}
```

期望输出：

```json
{
  "dtcs": ["P0300"],
  "raw": {},
  "source": "diagnostic_tool"
}
```

### `knowledge_query_dtc_solution`

查询单个 DTC 的维修手册证据。

建议输入：

```json
{
  "dtc": "P0300",
  "include_evidence": true,
  "include_diagnostic_checks": true
}
```

期望输出：

```json
{
  "dtc": "P0300",
  "description": "",
  "evidence": [],
  "possible_causes": [],
  "diagnostic_checks": [],
  "repair_actions": [],
  "confidence": 0.0,
  "review_status": "unreviewed"
}
```

### `knowledge_search`

当没有 DTC 时，按症状、部件、系统或关键词检索证据。

建议输入：

```json
{
  "query": "rough idle",
  "type": "all"
}
```

## 工作流

### Step 1：判断请求类型

识别用户是否提供：

- DTC
- 症状
- 车辆上下文
- 诊断工具输出
- 读取 DTC 的请求
- 维修或检查问题

### Step 2：检索证据

如果用户已经提供 DTC：

1. 规范化 DTC code。
2. 对每个 code 调用 `knowledge_query_dtc_solution`。
3. 保留证据引用，用于最终报告。

如果用户要求读取 DTC：

1. 调用 `diagnostic_connect`。
2. 连接成功后调用 `diagnostic_read_dtc`。
3. 对每个 DTC 调用 `knowledge_query_dtc_solution`。

如果没有 DTC：

1. 用症状或系统调用 `knowledge_search`。
2. 只有在证据检索无法继续时，才询问缺失车辆信息。

### Step 3：带边界地推理

明确区分：

- 用户描述
- 诊断工具结果
- 维修手册证据
- AI 推断
- 缺失证据

不要把推断写成事实。不要编造数值或流程。

### Step 4：输出报告

使用以下格式：

```markdown
# 车辆诊断报告

## 摘要

- 用户症状：
- 车辆上下文：
- DTC：
- 证据状态：

## 诊断发现

### DTC {{code}}

- 描述：
- 证据：
- 可能原因：
- 推荐检查：
- 维修动作：
- 置信度：
- 复核状态：

## 缺失信息

- ...

## 下一步

1. ...
2. ...
3. ...
```

## 安全规则

- 对 mock 或已授权来源，可以使用只读工具。
- 本社区工作流不清除 DTC。
- 本社区工作流不发送自定义 UDS 命令。
- 不执行动作测试、标定、自学习、刷写或写操作。
- 如果用户请求受限操作，说明该操作需要单独批准的安全流程。
- 如果证据不足，说明“当前知识库证据不足”，并建议下一步需要补充的数据。

## 严重程度建议

使用保守严重程度标签：

| 严重程度 | 含义 | 建议 |
|---|---|---|
| 安全关键 | 可能影响制动、转向、高压系统、安全气囊、火灾风险或行驶安全 | 停止并交由合格人员检查 |
| 高 | 可能损坏主要部件或造成不安全驾驶状态 | 尽快检查，避免不必要驾驶 |
| 中 | 影响性能、排放、舒适性或可靠性 | 按计划诊断和维修 |
| 低 | 历史码、间歇问题或证据不足 | 监测、收集更多数据并验证 |

除非证据支持，不要声称严重程度一定准确。

## 示例

用户：

```text
发动机故障灯亮了，诊断仪报 P0300。
```

Agent 行为：

1. 调用 `knowledge_query_dtc_solution` 查询 `P0300`。
2. 总结证据。
3. 只列出证据支持的可能原因。
4. 推荐检查步骤。
5. 标注证据缺口和复核状态。

