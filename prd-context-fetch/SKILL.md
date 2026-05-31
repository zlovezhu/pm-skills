---
name: prd-context-fetch
description: >
  PRD 撰写前置：根据 TAPD 需求 ID 或模块名，主动从 Obsidian 知识库（wiki + card 为主，raw 兜底）
  和 TAPD 历史需求中拉取相关材料，输出一份结构化的"参考清单"供用户勾选确认，再喂给 prd-writer。
  当用户说「写需求前先查一下知识库」「这个需求有相关参考吗」「拉一下这个模块的历史」「查 wiki」
  「先看下相关沉淀」，或在调用 prd-writer 前需要前置检索时，应触发此 Skill。
---

# PRD Context Fetch —— PRD 撰写前置检索

## 这个 Skill 解决什么

在真实的产品工作里，写 PRD 时常见的痛点是：
- 知识库里早就沉淀过相关 wiki 或 card，但容易忘了去查
- TAPD 里同模块的历史需求往往含已踩过的边界情况，但翻起来很慢
- 之前竞品调研的笔记散落在 Obsidian 里，每次写新需求都要重新找

这个 Skill 的职责：**给一个 TAPD 需求 ID 或模块名，主动跑一遍知识库 + 历史需求检索，把"相关度高的清单"摆到用户面前让 Ta 勾选，确认后把选中的材料喂给 `prd-writer`。**

核心定位：**主动检索 + 一次确认，不替用户做判断**。

## 适用场景

- 准备写一个新需求，想先看看是否有相关沉淀
- 评估一个需求时想快速 review 同模块历史
- 写 PRD 中途卡壳，想看看竞品笔记里有没有参考方案

## 核心规则

### 规则 1：检索范围按"三层 + 兜底"分级

```
默认范围：
  ✅ Obsidian: wiki/      （已沉淀的事实结论，PRD 撰写主依据）
  ✅ Obsidian: card/      （抽象后的原则，作为提示注入）
  ❌ Obsidian: raw/       （原始素材，默认排除）
  ✅ TAPD: 当前项目同模块需求

兜底降级：
  当 wiki/ 和 card/ 都返回空时
  → 自动降级到 raw/，并在结果里明确标注
    「以下来自 raw/，未沉淀，仅供参考」
```

这条规则是本 Skill 的核心约束："沉淀为 wiki 才有意义"——所以默认不让 raw 的未消化素材污染 PRD。

### 规则 2：模块标签每次现拉，不写死

不要在 Skill 里硬编一份模块词典。每次执行时：

1. 调 Obsidian MCP 拿当前 vault 里所有 tag
2. 把用户输入（TAPD 标题 / 模块名）和 tag 列表做语义匹配，给出 top 3 候选
3. 让用户勾选用哪些 tag 检索

理由：知识库会随时间长出新 tag，硬编会过时。

### 规则 3：TAPD 历史需求只查"当前项目 + 同模块"

不要查跨项目，不要查所有模块。检索条件：

```
project_id = 当前项目（<YOUR_PROJECT>）
module = 用户确认的模块（来自规则 2 的勾选结果）
status ∈ ['已发布', '已关闭', '开发中']  # 排除"已拒绝"
order by 修改时间 desc
limit 10
```

按"同模块 + 近 6 个月"是默认窗口，用户可调整。

### 规则 4：必须经过"一次确认"才进入下一步

绝对不要：
- 检索完直接把所有结果塞给 `prd-writer`
- 自动判断哪些有用、哪些没用

必须：
- 列出候选清单
- 标注每条的"相关度判断依据"
- 让用户勾选保留哪些
- 用户确认后才组装成 context 包

## 工作流程

### Step 1：识别检索目标

接收输入，识别成下面三类之一：
- TAPD 需求 ID（如 `1234567`）→ 先去 TAPD 拉这条需求的标题、模块、描述
- 模块名（如 `项目列表`、`文档管理`）→ 直接走规则 2 拿 tag
- 自然语言描述（如"想做一个文档批量导出的功能"）→ 提取核心模块词 + 走规则 2

### Step 2：拉取 Obsidian 当前 tag 列表

调 Obsidian MCP，取所有 tag。把 tag 和检索目标做语义匹配，输出 top 3 候选 tag。

如果只匹配到 1 个明显的，可以默认选中并跳到 Step 3，不浪费用户一次确认。

### Step 3：按规则 1 跑分层检索

并行跑：

```
A. wiki/ + 选中 tag        → wiki 命中清单（按相关度排序，top 5）
B. card/                   → card 命中清单（top 3，作为原则提示）
C. TAPD 同模块历史需求      → 历史需求清单（top 10）
D. （仅当 A+B 全空）raw/    → raw 兜底清单（top 3）
```

### Step 4：输出"参考清单"，让用户勾选

模板见 `references/fetch_result_template.md`。

清单里每条都标：
- 标题 / 链接
- 相关度判断依据（一句话，比如"标题命中关键词'批量导出'"）
- 默认勾选状态（强相关默认勾，弱相关默认不勾）

### Step 5：用户确认后组装 context 包

把用户保留的内容打包成结构化 context：

```yaml
target:
  type: story | bug
  module: 项目列表
  title: ...
context:
  wiki:
    - title: ...
      url: ...
      summary: ...   # 摘要 200 字以内
      excerpt: ...   # 关键段落原文
  card:
    - title: ...
      content: ...   # card 全文
  tapd_history:
    - id: ...
      title: ...
      key_points: ...  # 关键边界情况、已知约束
  raw_fallback:    # 仅当兜底降级时存在
    - ...
```

这份 context 直接喂给 `prd-writer` 或回给用户用。

## 输出风格

- **简洁**：清单形式，不啰嗦
- **可勾选**：每条带 `[ ]` checkbox 让用户标
- **有依据**：每条说明为什么相关（一句话）
- **明示降级**：如果走了 raw 兜底，开头就提示"未找到 wiki，已降级到 raw"

## 不做什么（边界）

- 不替用户判断"这条相关 / 不相关"——只给候选 + 依据
- 不主动写 PRD（那是 `prd-writer` 的事）
- 不修改 Obsidian 笔记（只读）
- 不爬全网竞品资料——只走 Obsidian 里已沉淀的内容；外部检索另起 Skill

## 与其他 Skill 的协作

- **上游**：`tapd-requirement-record` 落完单后可以直接接本 Skill
- **下游**：本 Skill 输出的 context 包直接喂给 `prd-writer`
- **互补**：如果检索发现"这个模块还没沉淀"，建议提示用户做 `case-distill` 把已知讨论沉淀进去再来

## 参考文件

- `references/fetch_result_template.md`：参考清单的输出模板
- `references/search_strategy.md`：分层检索的具体策略与排序规则

## 配置说明

使用前需要把以下占位符替换成你的实际值：

- `<YOUR_PROJECT>`：你的 TAPD 项目名称
- Obsidian vault 目录约定：本 Skill 假定你的 vault 采用 PARA 结构（wiki / card / raw 三层），如不一致请调整规则 1 的检索路径
