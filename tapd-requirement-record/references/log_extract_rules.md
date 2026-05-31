# 日志关键信息提取规则

> 给 `tapd-requirement-record` Skill 用。处理研发甩过来的日志文件时按这套规则走。

## 一、判定走哪种模式

```
读取日志大小 size
├── size < 200KB  → B 模式（关键提取）
├── size ≥ 200KB  → A 模式（简单引用）
└── 用户明确指定 → 按用户的来
```

## 二、A 模式：简单引用（大日志）

**目标**：日志只作为附件，不污染上下文，描述里只留一行索引。

**操作步骤**：
1. 不读完整内容，只用 `tail -n 200` / `head -n 100` 取首尾片段
2. 从片段里找一行最显眼的 ERROR 或 Exception
3. 描述里写：

   ```
   附日志：[文件名]（大小 [X] MB）
   关键报错（节选自日志末尾）：[ERROR 那一行]
   ```

4. 日志文件本体作为 TAPD 附件上传（如果 MCP 支持），否则在描述里给本地路径

**绝对不做**：
- 把整份大日志贴进描述
- 自己尝试"概括"一份没读过的日志
- 用模糊词如"看起来是 X 错误"

## 三、B 模式：关键提取（小日志）

**目标**：从日志里抽出 ERROR / 异常 / 关键时间戳 / 业务关键字，让 Bug 单的"实际效果"段直接可读。

### 3.1 优先级队列（按顺序找）

| 优先级 | 关键字 / 模式 | 处理方式 |
|---|---|---|
| P0 | `FATAL` / `panic` / `Segmentation fault` / `core dumped` | 必抽，整行 + 上下文 5 行 |
| P1 | `ERROR` / `Exception` / `Traceback` / `at .*\.kt:\d+` (Java/Kotlin 堆栈) | 必抽，整行 + 紧随的堆栈帧 |
| P2 | `WARN` / `failed` / `timeout` / `refused` | 选抽，只抽与本次问题相关的 |
| P3 | 业务关键字（按本项目"模块词典"，参见 `tapd_field_mapping.md` 第三节） | 抽与报错相邻的 1~2 行 |
| P4 | 时间戳：第一条报错的时间、最后一条报错的时间 | 必抽，用于"重现步骤"段 |

### 3.2 抽取后的渲染格式

写到缺陷描述的"实际效果"段：

```
## 实际效果

报错时间：2026-05-19 14:23:11 ~ 14:23:15

关键日志（来自 main-20260519-142311.log）：
\`\`\`
[14:23:11] ERROR [{ModuleName}] failed to load resource for {target}
    at com.example.{module}.{Class}.{method}({Class}.java:42)
    at com.example.{module}.{Caller}.{callerMethod}({Caller}.java:88)
    Caused by: java.io.FileNotFoundException: /tmp/{path}/{file}
[14:23:15] ERROR [{ModuleName}] processing aborted
\`\`\`

> 共抽取 2 条 ERROR、1 条堆栈，原日志见附件。
```

### 3.3 边界情况

- 日志里 ERROR 超过 20 条 → 只保留前 5 条 + 后 2 条，中间用 `... (省略 N 条)` 替代
- 日志全是 INFO / DEBUG，没有 ERROR → 在描述里诚实说明"未发现明显报错，请研发协助定位"，不要硬编错误
- 日志是非英文/非标准格式（如自研的二进制/特殊编码日志）→ 退回 A 模式

## 四、敏感信息脱敏

落单前必须扫一遍，以下内容要替换：

| 类型 | 替换成 |
|---|---|
| Token / API Key（形如 `sk-xxx`、`Bearer xxx`） | `[REDACTED_TOKEN]` |
| 内网 IP（10.* / 172.16-31.* / 192.168.*） | `[INTERNAL_IP]` |
| 用户手机号 / 邮箱 | `[REDACTED_CONTACT]` |
| 文件路径含用户名（如 `C:\Users\<user>\...`） | `C:\Users\<user>\...` |

如果不确定某段是不是敏感，**保守起见替换**，并在回执里提示用户"已对 N 处疑似敏感内容做脱敏，请核对"。

## 五、自检 checklist

抽取完日志后，Skill 自己跑一遍：

- [ ] 是否抽到了至少一条 ERROR / Exception？（如果没有，是否在描述里如实说明？）
- [ ] 时间戳是否提取了（用于"重现步骤"段）？
- [ ] 堆栈是否完整（不是只截了第一行）？
- [ ] 敏感信息是否脱敏？
- [ ] 原日志是否作为附件保留？
