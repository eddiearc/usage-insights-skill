---
name: usage-insights
description: 生成兼容 Claude Code 与 Codex CLI 的会话使用分析报告。用于"分析我的使用习惯/usage report/会话洞察/工作流复盘/生成美观报告"等场景；支持自动识别主要语言（中文/英文）并切换报告文案。
---

# Usage Insights

使用本技能生成「跨客户端（Claude Code + Codex CLI）」的统一历史复盘报告，并输出更美观的 HTML。

## 架构变更 (v2.0)

**移除了外部 API Key 依赖！** Agentic 分析现在由执行 skill 的 agent 自行完成：

1. **证据收集层**: Python 脚本从 Claude Code / Codex CLI 数据中提取结构化指标
2. **抽样复盘**: 自动抽取最近 20 个 session 的详细内容进行深度分析
3. **证据输出**: 生成 `*.evidence.json` 文件，包含统计数据 + 抽样样本
4. **Agentic 分析**: 由执行此 skill 的 agent 基于证据数据完成深度解读

## 抽样复盘 (新增)

脚本会自动抽取最近 **20 个 session** 的详细内容，包括：

### Claude Code 样本
- Session ID 和时间
- 首条 Prompt（前200字符）
- 用户消息数
- 使用的工具序列（前10个）
- 任务结果（outcome）
- 摩擦类型
- 项目路径

### Codex CLI 样本
- Session ID
- 消息总数
- 首条 Prompt（前200字符）
- 意图分类
- 是否有返工信号
- 是否有验证信号

这些数据让 agent 能够基于具体案例进行分析，而不是只看统计数据。

## 执行步骤

### 基础用法（生成报告 + 证据数据 + 抽样复盘）

```bash
python3 ./generate_usage_report.py --source auto --output ./artifacts/usage-insights-report.html
```

### 仅分析单一来源

```bash
# 仅 Claude Code
python3 ./generate_usage_report.py --source claude

# 仅 Codex CLI
python3 ./generate_usage_report.py --source codex
```

### 强制指定语言

```bash
python3 ./generate_usage_report.py --locale zh
python3 ./generate_usage_report.py --locale en
```

## 输出文件

- **HTML 报告**: 可视化报告，包含所有图表和指标
- **Evidence JSON**: `*.evidence.json`，包含：
  - 统计数据（metrics）
  - 趋势数据（trends）
  - 行为模式（patterns）
  - **抽样复盘数据（session_samples）**

### evidence.json 结构

```json
{
  "metrics": {
    "first_pass_rate": 0.72,
    "rework_rate": 0.18,
    "verification_rate": 0.25,
    "total_score": 78
  },
  "trends": {
    "recent_period": { "sessions": 45, "rework_rate": 0.15 },
    "previous_period": { "sessions": 38, "rework_rate": 0.22 }
  },
  "patterns": {
    "top_friction": [["误解需求", 12], ["代码缺陷", 8]],
    "top_intents": [["code_implementation", 35], ["debugging", 20]]
  },
  "session_samples": [
    {
      "source": "claude",
      "session_id": "xxx",
      "first_prompt": "帮我修复这个 bug...",
      "tool_sequence": ["read", "edit", "bash"],
      "outcome": "fully_achieved",
      "friction_types": ["misunderstood_request"]
    },
    {
      "source": "codex",
      "session_id": "yyy",
      "first_prompt": "重构这个函数...",
      "intent": "code_implementation",
      "has_rework_signal": true
    }
  ]
}
```

## Agentic 分析流程

执行此 skill 的 agent 应该：

1. 运行 Python 脚本生成报告和证据文件
2. 读取 `*.evidence.json` 中的结构化数据
3. **重点分析 `session_samples` 中的具体案例**
4. 基于统计数据 + 抽样案例生成深度洞察
5. 将分析结果补充到报告中或直接向用户呈现

### 分析要点

基于证据数据，agent 应该关注：

- **统计数据层面**: 
  - 高分维度如何进一步放大优势
  - 低分维度的具体改进步骤
  - 趋势变化（最近14天 vs 前14天）

- **抽样复盘层面**:
  - 高频摩擦类型的具体表现
  - 返工信号的常见场景
  - 工具使用模式是否合理
  - 会话长度与任务复杂度的关系

## 默认数据源

- Claude Code: `~/.claude/usage-data/`
- Codex CLI: `~/.codex/history.jsonl`

可通过参数覆盖：

```bash
python3 ./generate_usage_report.py \
  --claude-dir /custom/.claude/usage-data \
  --codex-history /custom/.codex/history.jsonl
```

## 报告内容

- 总览指标：会话数、消息数、活跃天数、主要语言
- 数据源覆盖：Claude Code + Codex CLI 分别统计
- 使用姿势评分：总分 + 5 维度分（每项 20 分）
- 深度诊断：6 个健康指标卡（健康/关注/风险）
- 趋势对比：最近 14 天 vs 前 14 天
- 关键洞察、优势、短板、后续方向
- 语言分布、活跃时段、意图分布
- 高频工具、任务结果、摩擦点、项目路径
- Agentic 解读区块（由执行 agent 基于证据数据填充）

## "更好的使用方式"判定标准

- 一次达成率更高：同等任务下，最终通过次数更多、返工更少
- 返工率更低：`still/again/返工` 等信号下降
- 验证更早：在完成前明确触发 `test/lint/verify/回归`
- 切换更少：同类任务固定主链路，工具跳转减少
- 趋势更稳：最近 14 天相对前 14 天，返工下降且验证覆盖提升
