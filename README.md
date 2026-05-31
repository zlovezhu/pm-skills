# AI PM Workflow Skills

一套给 AI Agent 用的产品经理工作流 Skill 集合。把"零散输入"（IM 聊天截图、会议纪要、研发日志、散记）变成"结构化产出"（TAPD 工单、PRD、Wiki、Card），减少产品经理的搬运工作量。

兼容 Claude Code、WorkBuddy 以及任何遵循 `SKILL.md + references/` 结构的 Agent Skill 体系。

---

## 包含的 4 个 Skill

| Skill | 干什么的 | 典型触发语 |
|---|---|---|
| **tapd-requirement-record** | 把零散输入结构化成 TAPD 需求/缺陷单 | "把聊天里这个录到 TAPD"、"提个 bug"、"日志里这个问题建个单" |
| **prd-context-fetch** | PRD 撰写前置检索，从知识库 + 历史需求拉相关材料 | "写需求前查一下知识库"、"这个模块有相关参考吗" |
| **prd-writer** | 按四级编号体系起草 PRD，支持版本管理 + 可选 HTML 低保真原型 | "写需求"、"PRD"、"产品需求文档" |
| **case-distill** | 把问题排查/方案讨论沉淀成 Wiki + 0~3 张 Card | "把这个排查记下来"、"沉淀一下"、"做个复盘" |

四个 Skill 可以独立使用，也能串起来跑一条完整链路：

```
零散输入 ──tapd-requirement-record──▶ TAPD 单
                                       │
                                       ▼
              prd-context-fetch ──▶ 相关参考 ──▶ prd-writer ──▶ PRD（+ 可选 HTML 原型）
                                                                    │
                                       上线后排查/复盘 ──▶ case-distill ──▶ Wiki + Card
```

---

## 目录结构

```
skills-public/
├── tapd-requirement-record/
│   ├── SKILL.md                       # Skill 入口，含触发条件、主流程、字段判断逻辑
│   └── references/
│       ├── tapd_field_mapping.md      # TAPD 需求/缺陷字段模板与默认值规则
│       ├── log_extract_rules.md       # 研发日志的提取规则（按大小分档处理）
│       └── output_template.md         # 给用户看的回执模板
│
├── prd-context-fetch/
│   ├── SKILL.md
│   └── references/
│       ├── search_strategy.md         # 检索范围、过滤规则、tag 优先级
│       └── fetch_result_template.md   # 参考清单的输出格式
│
├── prd-writer/
│   ├── SKILL.md
│   └── references/
│       ├── prd_example.md             # 完整 PRD 范例（四级编号体系）
│       ├── version_table_template.md  # 版本表 + "读旧版→追加变更行→改正文" 流程
│       └── html_prototype_template/   # 可选 HTML 低保真原型脚手架（pf- class 规范）
│
└── case-distill/
    ├── SKILL.md
    └── references/
        ├── raw_template.md            # 原始素材保留模板
        ├── wiki_template.md           # Wiki 五段式：发现异常→看现场→还原过程→找热点→落到代码
        └── card_template.md           # Card 三问法判断 + 标准结构
```

每个 Skill 都是 **`SKILL.md`（入口） + `references/`（被入口按需引用的细则）** 的两层结构，Agent 加载时只读 `SKILL.md`，需要细则时再 follow link 进 `references/`，避免一次性塞太多上下文。

---

## 安装到 Claude Code

Claude Code 默认会加载 `~/.claude/skills/` 下的所有 Skill（用户级），或当前项目下 `.claude/skills/` 的 Skill（项目级）。

### 方式一：克隆后软链 / 复制（推荐）

```bash
# 1. 把 repo clone 到本地任意位置
git clone https://github.com/zlovezhu/pm-skills.git
cd pm-skills

# 2. 复制 4 个 skill 到 Claude Code 的用户级 skills 目录
#    macOS / Linux
mkdir -p ~/.claude/skills
cp -r tapd-requirement-record prd-context-fetch prd-writer case-distill ~/.claude/skills/

#    Windows (PowerShell)
mkdir $HOME\.claude\skills -Force
Copy-Item -Recurse tapd-requirement-record, prd-context-fetch, prd-writer, case-distill $HOME\.claude\skills\
```

只装其中一个也行，按需复制对应子目录即可。

### 方式二：仅在当前项目启用

```bash
# 在你的项目根目录
mkdir -p .claude/skills
cp -r /path/to/pm-skills/prd-writer .claude/skills/
```

### 验证安装

启动 Claude Code，输入触发语：

```
/skill                     # 查看已加载的 skill 列表
帮我把这段聊天记录提个 TAPD 需求    # 触发 tapd-requirement-record
帮我写一份 PRD，关于 XXX           # 触发 prd-writer
```

如果 Claude Code 正确识别，会在回复前提示 "Loading skill: tapd-requirement-record"。

> **WorkBuddy 用户**：把目标路径换成 `~/.workbuddy/skills/` 即可，其他步骤一致。

---

## 怎么用

### 1. tapd-requirement-record —— 提 TAPD 单

**输入**：任何半成品

```
用户：把这段录成 TAPD 缺陷
[贴一张企微截图或一段日志文本]
```

**Skill 行为**：
- 自动判断是需求还是缺陷
- 按字段模板把能推断的都填好（标题、模块、描述、重现步骤……）
- 必填项缺失才反问（比如优先级、迭代）
- 日志超过 200KB 时只做关键提取，不全量塞描述
- 输出一段可直接走 TAPD MCP 创建的结构化 JSON + 一份给你看的简洁回执

**适合接的下游**：TAPD MCP / TAPD OpenAPI。

> **不用 TAPD 也能跑**：本 Skill 默认对接 TAPD，但工作流逻辑（判断单类型 / 字段模板 / 日志分档 / 回执格式）跟具体平台无关。如果你团队用 Jira、禅道、Worktile、PingCode、GitHub Issues 等其他需求管理工具，把 `references/tapd_field_mapping.md` 里的字段名和取值对照换成对应工具的字段，再把"适合接的下游"换成对应工具的 API 或 MCP 即可。后续会抽出一个 `field_mapping.local.md` 让换工具更方便。

### 2. prd-context-fetch —— PRD 撰写前置检索

**输入**：要写的需求大致主题或 TAPD 单 ID

```
用户：我要写 XXX 模块的 PRD，先查一下知识库有没有相关参考
```

**Skill 行为**：
- 默认检索范围：本地知识库（Obsidian / 任意 Markdown 仓库）的 wiki + card 层
- 同步查 TAPD 当前项目近 6 个月的同模块历史需求
- raw 笔记默认排除，找不到时才兜底
- 输出一份「参考清单」，每条带链接 + 一句话摘要 + 推荐相关度
- 强制走"一次确认"再交给 prd-writer

**适合接的下游**：prd-writer。

### 3. prd-writer —— 起草 PRD

**输入**：需求描述（可以来自 prd-context-fetch 的输出，也可以直接写）

```
用户：写一份 PRD，关于「XXX 功能」
```

**Skill 行为**：
- 按四级编号体系起草（1. / 1.1 / 1.1.1 / 1.1.1.1）
- 头部强制带版本表（版本 / 更新时间精确到分钟 / 改动摘要 / 影响范围 / 修改人）
- 改 PRD 时强制走「读旧版 → 追加变更行 → 用户确认 → 改正文」，不允许跳步
- 可选输出 HTML 低保真原型（用户明说才触发，3~5 个核心流程页，pf- class 规范）
- 支持双向同步：原型上的产品决策类改动可回流到 PRD，纯视觉调整不动 PRD

**适合接的下游**：把 PRD Markdown 提交到知识库 / TAPD wiki / 飞书；HTML 原型给设计师做高保真参考。

### 4. case-distill —— 复盘沉淀

**输入**：一段排查过程、方案讨论记录、上线复盘

```
用户：把这次排查记下来，做个 case
[贴讨论记录或排查日志]
```

**Skill 行为**：
- 先保留 raw 副本（不丢原始信息）
- 输出一份 Wiki，按五段式组织：发现异常 → 看现场 → 还原过程 → 找热点 → 落到代码
- 用三问法判断要不要拆 Card（0~3 张），每张 Card 是一个可独立复用的小结论
- 默认落地到 Obsidian Vault（或你指定的任意 Markdown 知识库）

**适合接的下游**：Obsidian / 任意支持 Markdown 的知识库。

---

## 本地化适配

这套 Skill 在写的时候是基于一个具体项目的语境，所以有些常量是写死的。fork 后第一次跑之前，建议先 grep 一下下面这些字段，按你团队的实际情况替换：

| 类别 | 在哪些文件里 | 怎么改 |
|---|---|---|
| 模块名 | 各 `SKILL.md`、`tapd_field_mapping.md` | 改成你产品的实际模块划分 |
| TAPD 项目 ID / 字段配置 | `tapd-requirement-record/references/tapd_field_mapping.md` | 改成你团队的 TAPD workspace_id 和字段映射 |
| 知识库目录结构 | `prd-context-fetch/references/search_strategy.md`、`case-distill/references/*.md` | 默认按 PARA + 三层（raw/wiki/card），不用 PARA 的话改成你自己的目录 |
| 默认处理人 | `tapd-requirement-record/SKILL.md` | 默认填当前用户，要改成别人需要在 SKILL.md 里调 |

后续会把这些拆成一个独立的 `config.local.md`，目前先手动 grep。

---

## 路线图

- [ ] 抽出占位符体系，把项目相关常量集中到一个 `config.local.md`
- [ ] 增加 `prd-edge-case-augment`（PRD 边界情况自动补全）
- [ ] 增加 `acceptance-tracker`（验收清单 + 状态跟踪）
- [ ] 提供一个最小可跑的 demo（不依赖任何内部系统）

---

## License

MIT。随便用、随便改、随便商用，但不附带任何担保。详见 [LICENSE](./LICENSE)。

---

## 反馈

发现 Bug、有改进建议、想分享你 fork 后的版本，欢迎开 Issue 或 PR。
