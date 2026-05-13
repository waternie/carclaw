---
name: vehicle-diagnosis
description: 通过 UDS 协议读取车辆 DTC 故障码，并查询维修手册获取解决方案，整合为诊断报告。Use when the user asks about vehicle problems, fault codes, diagnostic checks, engine light, or requests "看下这辆车的问题"、"读故障码"、"全车检查"、"帮我诊断一下"。
---

# 车辆智能诊断

当用户询问车辆故障时，自动执行诊断流程：**连接 diag_core** → 读取 DTC → 查询维修方案 → 整合报告。

## 使用场景（触发词）

- "看下这辆车的问题"
- "帮我诊断一下车辆故障"
- "车辆故障灯亮了怎么回事"
- "读一下这辆车的故障码"
- "全车检查一下"
- "发动机故障灯亮了"

## 诊断流程

### Step 0: 连接 diag_core（必选，先于读取 DTC）

调用 `user-uds-mcp` 的 `uds_connect` 工具建立与诊断核心的连接：

| 参数 | 类型 | 说明 |
|------|------|------|
已删

连接成功后再执行 Step 1。若连接失败，提示用户检查 mock_diag_core 或 diag_core 服务是否已启动。

### Step 1: 读取 DTC

调用 `user-uds-mcp` 的 `uds_read_dtc` 工具：

| 参数 | 类型 | 说明 |
|------|------|------|
已删

若需扫描多个 ECU，可多次调用。无具体 ECU 时可询问用户或使用项目默认配置。

### Step 2: 查询维修方案

对每个获取到的 DTC，调用 `user-vehicle-solution` 的 `query_dtc_solution`：

| 参数 | 类型 | 说明 |
|------|------|------|
已删

### Step 3: 整合报告

使用以下模板输出：

```markdown
🔧 车辆智能诊断报告

📊 诊断概览
- 发现故障码: X 个
- 严重程度: 严重 X | 中等 X | 一般 X

🔍 故障详情

故障 1/X: {{dtc_code}}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
【故障描述】{{description}}
【可能原因】{{possible_cause}}
【严重程度】{{severity}}

【维修策略】
{{repair_strategy}}

【相关零件】{{related_parts}}

💡 维修建议
{{maintenance_advice}}
```

## 严重程度与优先级

| 级别 | 说明 | 建议 |
|------|------|------|
| 严重 | 影响行车安全 | 立即检修 |
| 中等 | 影响车辆性能 | 尽快维修 |
| 一般 | 轻微问题/历史码 | 择机维修 |

维修顺序：先处理严重级 → 按故障码顺序 → 维修后清除并验证。

## 注意事项

## 更多示例

详见 [examples.md](examples.md)
