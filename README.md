# AI PM Workflow Skills

一套给 AI Agent 用的产品经理工作流 Skill 集合。把"零散输入"（聊天截图、会议纪要、研发日志、散记）变成"结构化产出"（TAPD 工单、PRD、Wiki、Card），减少产品经理的搬运工作量。

> 适配 [WorkBuddy](https://www.codebuddy.cn/docs/workbuddy/Overview) / Claude Skills 体系；底层是 SKILL.md + references/ 的标准结构，理论上任何支持 Skill 加载的 Agent 都能跑。

---

## 包含的 4 个 Skill

| Skill | 干什么的 | 触发场景 |
|---|---|---|
| **tapd-requirement-record** | 把零散输入结构化成 TAPD 需求/缺陷单 | "把聊天里这个录到 TAPD"、"提个 bug"、"日志里这个问题建个单" |
| **prd-context-fetch** | PRD 撰写前置检索，从知识库 + 历史需求拉相关材料 | "写需求前查一下知识库"、"这个模块有相关参考吗"、"先看下相关沉淀" |
| **prd-writer** | 按四级编号体系起草 PRD，支持版本管理 + 可选 HTML 低保真原型 | "写需求"、"PRD"、"产品需求文档" |
| **case-distill** | 把问题排查/方案讨论沉淀成 Wiki + 0~3 张 Card | "把这个排查记下来"、"沉淀一下"、"做个复盘" |

每个 Skill 的详细说明见对应目录下的 `SKILL.md`。

---

## 设计原则

这 4 个 Skill 共享一套设计原则，能解释它们为什么长这样：

1. **主动做事，但保留一次确认。** Agent 默认能推断的字段直接填，只在真的缺关键信息时反问；执行前给用户一次"要不要这么干"的确认机会。
2. **不替用户做主观判断。** 业务价值、需求来源、优先级判定这类需要业务背景的字段，一律留空让用户自己填，不要瞎猜。
3. **输出风格直白，不堆砌。** 不写"在快速变化的市场环境下"这种废话，所有产出都过一遍"去 AI 味"检查。
4. **流程强制性 > 灵活性。** 比如改 PRD 必须走"读旧版 → 追加变更行 → 用户确认 → 改正文"，不允许跳步——因为跳步出问题的概率比省事的收益大。

---

## 怎么用

### 安装到 WorkBuddy（推荐）

把这个 repo clone 到本地，把需要的 Skill 子目录复制到 `~/.workbuddy/skills/`：

```bash
git clone https://github.com/<你的用户名>/<repo名>.git
# Windows
xcopy /E /I skills-public\tapd-requirement-record %USERPROFILE%\.workbuddy\skills\tapd-requirement-record
# macOS / Linux
cp -r skills-public/tapd-requirement-record ~/.workbuddy/skills/
```

重启 WorkBuddy，Skill 就会被自动识别。

### 适配自己的项目（重要）

这套 Skill 里有不少地方是基于 **Perflame**（一个游戏性能分析平台）的语境写的，比如：

- 模块名：工作台 / 采集器 / 符号表 / 火焰图 / 源码视图 / 线程分析 ……
- TAPD 项目 ID、字段配置
- 知识库目录结构（Obsidian + PARA）

如果你不是 Perflame 团队的，需要先做本地化：

1. 通读对应 Skill 的 `SKILL.md` 和 `references/`，找到所有 `Perflame` 相关字眼
2. 替换成你自己项目的模块名、TAPD ID、知识库路径
3. 字段模板、状态枚举值、优先级口径，按你团队约定改

后续我会把"哪些地方需要本地化"做成专门的占位符体系，目前还是手动 grep。

---

## 路线图

- [ ] 抽出"占位符体系"，把项目相关常量集中到一个 `config.local.md`，方便 fork 时一处替换
- [ ] 增加 `prd-edge-case-augment`（PRD 边界情况自动补全）
- [ ] 增加 `acceptance-tracker`（验收清单 + 状态跟踪）
- [ ] 提供一个最小可跑的 demo 项目（不依赖任何内部系统）

---

## License

MIT。随便用、随便改、随便商用，但不附带任何担保。详见 [LICENSE](./LICENSE)。

---

## 反馈

发现 Bug、有改进建议、想分享你 fork 后的版本，欢迎开 Issue 或 PR。
