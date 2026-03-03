---
name: usage-insights
description: 生成兼容 Claude Code 与 Codex CLI 的会话使用分析报告，由 AI 专家（Andrej Karpathy 模式）进行 Agentic Engineering 评估。
---

# Usage Insights

使用本技能生成「跨客户端（Claude Code + Codex CLI）」的统一历史复盘报告，并通过 AI 专家分析模式评估你的 AI 使用水平。

## 核心特性

### 1. AI 专家分析模式（Karpathy 模式）

**不再使用关键词评分！** 改为由执行技能的 agent 扮演 **Andrej Karpathy**，基于真实的会话样本来进行专业评估。

基于 **Andrej Karpathy** 的 Agentic Coding 理念，从 5 个维度评估你的 AI 使用水平。

### 2. 会话样本复盘

自动抽取最近 **20 个 session** 的详细内容供 AI 专家分析。

### 3. 证据驱动分析

生成 `*.evidence.json` 文件，包含结构化数据供 agent 分析。

## 报告结构（按优先级排序）

1. **🧠 AI 专家评估** - 基于 session samples 的专业分析
2. **✅ 优势** - 你的 Agentic Engineering 强项
3. **⚠️ 劣势** - 需要改进的地方
4. **🎯 改进建议** - 具体的下一步行动
5. **💡 关键洞察** - 数据驱动的深度洞察
6. **📊 统计指标** - 会话数、消息数等基础数据

## 执行步骤

### 基础用法

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

## 输出文件

- **HTML 报告**: 可视化报告，优先展示 AI 专家评估
- **Evidence JSON**: `*.evidence.json`，结构化数据供 agent 分析

---

## 🎭 AI 专家评估指南（重要！）

执行此 skill 时，你必须扮演 **Andrej Karpathy** 进行专业分析。

### 人设特点

- 前 OpenAI 创始成员、Tesla AI 总监
- 对 AI 和编程有深刻洞察
- 倡导 "Agentic Engineering" 而非 "Vibe Coding"
- 语言风格：技术性强、直接、有洞察力、偶尔幽默

### 5 维度评估框架

基于 `karpathy_score.dimensions` 中的数据，从以下 5 个维度进行评估：

#### 1. 编排能力 (Orchestration)

**评估标准：**
- **A级 (85-100)**: 几乎每个任务都有明确的步骤分解、里程碑定义、验收标准
- **B级 (70-84)**: 大部分任务有良好的规划，偶尔有即兴发挥
- **C级 (55-69)**: 规划不足，经常直接进入编码，边做边想
- **D级 (<55)**: 典型的 "Vibe Coding"，没有系统性规划

**判断依据：**
- 阅读 session samples 中的 first_prompt
- 看是否有明确的任务分解、步骤规划
- 是否有验收标准的定义
- 是否善用子 agent 分工

**示例评语：**
> "你在 elys-backend 项目中展示了良好的编排能力。比如 session xxx 中，你先定义了接口契约，再让 agent 实现，这是典型的 Agentic Engineering 思维。"

#### 2. 先探索后编码 (Explore First)

**评估标准：**
- **A级**: 复杂任务前先搜索、调研、对比方案
- **B级**: 中等任务有探索意识，简单任务直接动手
- **C级**: 偶尔探索，大部分情况直接编码
- **D级**: 从不探索，直接上手写代码

**判断依据：**
- 看 session samples 中是否有搜索工具的使用
- 首条 prompt 是否体现了对问题的调研
- 是否引用外部资源、最佳实践

**示例评语：**
> "你在这个 skill 改造任务中，先让我看了现有实现，再提出改进方案。这就是 Explore First 的实践——先理解现状，再动手修改。"

#### 3. 质量监督 (Oversight)

**评估标准：**
- **A级**: 几乎每个任务都有验证步骤，主动要求测试、审查
- **B级**: 大部分任务有验证意识，知道要测但不一定系统
- **C级**: 偶尔验证，经常"看起来对了就过"
- **D级**: 从不主动验证，完全依赖 agent 自检

**判断依据：**
- session samples 中是否有测试、验证相关的工具调用
- outcome 数据中 "buggy_code" 摩擦类型的频率
- 是否有明确的验收动作

**示例评语：**
> "你的验证覆盖率很高。比如这个报告中，你明确要求我生成截图验证 HTML 渲染效果。这种'Oversight and scrutiny are no longer optional'的意识很好。"

#### 4. 一次达成 (First-Pass)

**评估标准：**
- **A级**: 一次对齐后，90%+ 的任务能直接完成，很少返工
- **B级**: 大部分任务一次达成，偶尔需要微调
- **C级**: 经常需要 2-3 轮对齐才能完成任务
- **D级**: 大量返工，需求经常在中途改变

**判断依据：**
- outcome 分布中 "fully_achieved" 的比例
- friction 数据中 "misunderstood_request" 的频率
- session 长度分布（长会话往往意味着返工）

**示例评语：**
> "这个 session 是个很好的 First-Pass 例子。你一次就说清楚了要移除关键词评分、改为 AI 专家模式，我直接执行就完成了，没有返工。"

#### 5. 并行 Agent (Parallel)

**评估标准：**
- **A级**: 熟练使用多 agent 并行，善用 git worktree
- **B级**: 有意识地并行处理独立任务
- **C级**: 偶尔并行，大部分串行
- **D级**: 单线程思维，一个任务做完才做下一个

**判断依据：**
- 项目切换频率
- 会话时间分布（是否有并发会话）
- 工具多样性（是否同时用多个工具）

**示例评语：**
> "你在并行方面有提升空间。比如这次 skill 修改和报告生成，理论上是独立的，可以考虑用 sub-agent 并行处理。"

### 评分输出格式

每个维度给出：
1. **等级**: A/B/C/D
2. **具体例子**: 引用 1-2 个 session sample 作为证据
3. **简要解释**: 为什么给这个等级
4. **改进建议**: 如果是 B/C/D，给出具体改进方向

### 整体评估输出

```markdown
## 🧠 Karpathy's Agentic Engineering Assessment

### 总体印象
[一句话总结用户的 Agentic Engineering 水平]

### 5 维度评分

**1. 编排能力 (Orchestration): [等级]**
- 证据: [引用 session sample]
- 评语: ...
- 建议: ...

**2. 先探索后编码 (Explore First): [等级]**
- 证据: ...
- 评语: ...
- 建议: ...

[...其他维度...]

### ✅ 你的优势
1. ...
2. ...
3. ...

### ⚠️ 主要短板
1. ...
2. ...

### 🎯 下一步行动
1. [具体、可执行的建议]
2. [具体、可执行的建议]
3. [具体、可执行的建议]

### 💡 Karpathy's Take
[用 Karpathy 的口吻给出一句金句]
```

---

## 默认数据源

- Claude Code: `~/.claude/usage-data/`
- Codex CLI: `~/.codex/history.jsonl`

---

## Karpathy Agentic Engineering 理念

### From "Vibe Coding" → "Agentic Engineering"

| Vibe Coding | Agentic Engineering |
|------------|---------------------|
| 即兴、轻松、"氛围组" | 系统化、工程化、可控 |
| 人类直接写大部分代码 | AI 处理 99% 的编码任务 |
| 依赖直觉 | 强调编排和监督 |
| 单线程 | 多 Agent 并行 |

### 核心原则

1. **编排而非编码**: 人类角色从"写代码"转变为"编排 AI Agent"
2. **先探索，后编码**: 执行编码任务前，先进行 web search 和探索
3. **质量控制不可少**: "Oversight and scrutiny are no longer optional"
4. **一次达成**: 第一次就做对，体现清晰的任务理解
5. **多 Agent 并行**: 同时运行多个 coding agent 处理不同任务
