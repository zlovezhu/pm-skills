# HTML 低保真原型模板说明

本目录是 PRD Writer Skill 在 C 分支（用户明说要 HTML 原型）触发时使用的起点模板。

## 触发方式

**只在用户明说时使用**：
- ✅「帮我生成 HTML 原型」
- ✅「画个原型」「出个可点击 demo」
- ✅「同步生成 HTML」
- ❌ 不要主动建议、不要默认生成

## 复制规则

生成原型时：

1. 在 PRD 同目录创建 `prototype/` 子目录
2. 复制 `styles.css` 到 `prototype/styles.css`
3. **设计规范对齐（重要，C 分支必走一步）**：问用户有没有团队设计规范；
   - 有 → 按 `design_tokens_example.md` 的结构提取最小集，落到 `prototype/design_tokens.md`，并刷 `styles.css` 顶部 `:root` 变量
   - 没有 → 用模板默认配色，跳过这一步
4. 基于 `_page_template.html` 为每个核心页生成一个 HTML
5. 生成 `index.html` 导航页串起所有原型页

## 文件清单

- `styles.css`：共享样式，灰白配色 + 简单布局；顶部用 CSS 变量挂主色 / 字号 / 控件高度等，方便对接团队设计规范
- `_page_template.html`：单个页面模板
- `_index_template.html`：导航页模板
- `design_tokens_example.md`：**设计规范对齐参考样本**——展示 AI 应该按什么粒度从用户提供的规范图里提取 token，作为 `prototype/design_tokens.md` 的写法参考

## 风格底线

- 灰白配色为主（`#fafafa` 背景、`#333` 文字、`#0066cc` 主色）
- 不写花哨动画、渐变、阴影
- 不引外部字体（用系统默认）
- 留给设计同学在 Figma 阶段做视觉

## 命名约定

class 命名统一前缀 `pf-`（prototype 的缩写，亦可按本地项目改成自己的前缀，如 `xx-`）：

| class | 用途 |
|-------|------|
| `.pf-page` | 页面容器 |
| `.pf-header` | 顶部标题/操作区 |
| `.pf-toolbar` | 筛选/搜索工具栏 |
| `.pf-card` | 卡片块 |
| `.pf-table` | 列表表格 |
| `.pf-button-primary` | 主按钮 |
| `.pf-button-secondary` | 次按钮 |
| `.pf-button-danger` | 危险按钮（删除等） |
| `.pf-modal` | 弹窗 |
| `.pf-modal-overlay` | 弹窗遮罩 |
| `.pf-empty` | 空状态 |
| `.pf-pagination` | 分页 |
| `.pf-tag` | 状态标签 |

> **本地化提示**：如果你的项目有自己的 class 前缀约定，建议在 fork 后用 IDE 全局替换 `pf-` → 你的前缀，并同步改 `styles.css` 头部注释。

## 每个 HTML 顶部必须有的注释

```html
<!--
  @prd: feat-2-1 项目列表页
  @prd-file: prd-project-list.md
  @last-sync: v1.2 / 2026-05-19
-->
```

修改 PRD 对应章节时，必须同步更新 `@last-sync`。
