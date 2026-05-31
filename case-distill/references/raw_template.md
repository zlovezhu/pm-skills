# Raw 副本规范

> 给 `case-distill` Skill 用。每次 distill 都要保留一份 raw 副本，作为"未消化的原始素材"。

## 一、为什么要保留 raw

wiki 是"已消化的结论"——经过 Skill + 用户判断后的提炼版本。
但提炼必然损失信息，几个月后想验证 wiki 是否还成立、想做二次提炼时，必须能回到原始上下文。

所以每次 distill：
- **wiki**：精炼、可读、收敛
- **raw**：原样、无损、可追溯
- 两者通过 frontmatter 双向链接（wiki 里 `related_raw`，raw 里 `related_wiki`）

## 二、Raw 文件落地位置

`0-Inbox/raw/{YYYY-MM-DD}-{slug}-raw.md`

slug 命名与 wiki 保持一致，只在末尾加 `-raw` 后缀，便于一眼区分。

## 三、Frontmatter 规范

```yaml
---
type: raw
date: 2026-05-19
source:
  - 团队聊天工具截图
  - 个人散记
  - main-20260519-142311.log
related_wiki: 2026-05-19-排查-{slug}.md
processed: true       # 是否已经被 distill 过
---
```

## 四、Raw 内容规范

**核心原则：原样保留，不加工**。

按下面结构组织（按来源分块），每块原样粘贴：

```markdown
# Raw · {主题}

## 来源 1：{聊天截图，发言人 {同事昵称}，2026-05-19 14:23}

> {同事昵称}：{某子特性}模式下操作之后取消按钮点了没反应
> 我：复现一下？
> {同事昵称}：必现，已附日志

## 来源 2：日志片段（main-20260519-142311.log，节选）

```
[14:23:11] ERROR [{ModuleName}] cancel invoked but state is INVALID
[14:23:15] ERROR [{ModuleName}] operation still active after cancel
...
```

## 来源 3：我的散记（2026-05-19 14:50）

排查思路：
- 第一反应：按钮事件没绑？
- 但 React DevTools 看到 onClick 触发了
- 看日志才发现状态机问题
- 14:50 收敛到 cancel 事件没在状态机注册

## 关联 wiki

→ 已沉淀到 [[2026-05-19-排查-{slug}]]
```

## 五、对 raw 的"不动"承诺

- **不修正**：raw 里的错别字、表达不清都保留，因为那是真实场景的样子
- **不补全**：raw 里缺的信息就缺着，不要事后回填——回填的内容应该进 wiki
- **不删除**：哪怕 raw 看起来"没用"也不删；几个月后可能就有用

唯一允许做的"加工"：
- 添加 frontmatter
- 给每块来源加一行 `## 来源 X：{描述}` 标头
- 对敏感信息（token、内网 IP、用户手机号）做脱敏，并在末尾备注"⚠️ 已脱敏 N 处"

## 六、Raw 与 wiki 的链接维护

每次 distill 完成后：

1. wiki 的 frontmatter `related_raw` 字段要指向 raw 文件名
2. raw 的 frontmatter `related_wiki` 字段要指向 wiki 文件名
3. raw 文末追加一行 `→ 已沉淀到 [[wiki 标题]]`
4. raw 的 `processed` 字段从 `false`（如果是预先收集的）改成 `true`

这样后续在 Obsidian 里点任何一篇都能跳到另一篇。
