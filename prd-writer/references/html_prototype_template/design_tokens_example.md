# 设计规范对齐 —— 参考样本

> 这份文件是**示例**，不是模板。
>
> 用途：给 PRD Writer 的 C 分支做参考——当用户提供团队设计规范（设计稿截图 / 组件库截图 / token 表）时，AI 应该把规范"翻译"成下面这种最小集格式，落到 `prototype/design_tokens.md`，并同步刷 `styles.css` 顶部变量。
>
> 下面是基于一个典型 B 端平台规范（按钮 / 字体 / 表单 / Dialog）整理出来的样本，方便你看清"应该提取到什么粒度"。

---

## 来源
- 设计规范截图 1：基础组件（按钮、字体、表单、下拉、面包屑、Tag）
- 设计规范截图 2：Dialog 对话框
- 提取时间：YYYY-MM-DD
- 用途：本次 PRD 配套 HTML 原型的视觉变量

## 颜色

| 变量 | 值 | 用途 |
|---|---|---|
| 主色 Primary | `#1664FF` | 主按钮、链接、当前激活状态 |
| 危险色 Danger | `#F53F3F` | 删除按钮、错误提示 |
| 文字主 | `#1D2129` | 标题、正文 |
| 文字次 | `#4E5969` | 辅助说明、表单 label |
| 文字辅 | `#86909C` | 占位符、弱提示 |
| 文字禁 | `#C9CDD4` | 禁用态文字、"暂时不上"灰 |
| 背景页 | `#F7F8FA` | 页面底色 |
| 背景卡片 | `#FFFFFF` | 卡片、Dialog、表单容器 |
| 分割线 | `#E5E6EB` | 表格分割线、卡片边框 |

## 字体

- **字体家族**：`HarmonyOS Sans, PingFang SC, Microsoft YaHei, sans-serif`
- **字号梯度**（中文）：
  - 大标题：16px / 22px line-height / 加粗
  - 小标题：14px / 22px line-height / 加粗
  - 正文：14px / 22px line-height
  - 辅助：12px / 18px line-height

> 字号档位**不要超过 4 档**。原型阶段不区分 H1/H2/H3 这种细分。

## 控件尺寸

| 控件 | 高度 | 圆角 | 备注 |
|---|---|---|---|
| 按钮（默认） | 32px | 4px | 内边距左右 16px |
| 按钮（小） | 24px | 4px | 列表行内操作用 |
| 输入框 | 32px | 4px | 与按钮同高，方便对齐 |
| 下拉框 | 32px | 4px | 同上 |
| Tag | 22px | 2px | 状态标签 |

## 间距

- 页面主内边距：24px
- 卡片内边距：16px
- 卡片间距：16px
- 表单字段间距：16px（垂直）/ 24px（水平）

## Dialog 对话框（来自规范截图 2）

按尺寸分四档，原型里用得到的主要是中尺寸：

| 类型 | 宽度 | 用途 |
|---|---|---|
| 极简提示 | 360px | 不可逆操作的二次确认（"确认删除 xxx 吗？"） |
| 短文案 | 480px | 一段说明 + 取消/确认 |
| 长文案 | 560px | 多行说明 |
| 表单/向导 | 720px | 多字段填写、分步流程（如"创建新项目"） |

通用结构：
- 顶部标题区：高 56px，左标题、右关闭按钮 ×
- 中间内容区：内边距 24px
- 底部按钮区：高 56px，按钮右对齐，主按钮在右、次按钮在左

## 待补 / 看不清的项

- {如截图里某项无法确认，列在这里，标 TODO，用默认值占位}
- 例：图标体系（截图未给出完整图标库）→ 原型里用 emoji 或文字占位

---

## 对应的 styles.css 顶部变量

把上面的 token 换成 CSS 变量挂在 `:root` 上，原型里只用变量、不直接写颜色值：

```css
:root {
  /* 颜色 */
  --pf-color-primary: #1664FF;
  --pf-color-danger: #F53F3F;
  --pf-color-text-primary: #1D2129;
  --pf-color-text-secondary: #4E5969;
  --pf-color-text-tertiary: #86909C;
  --pf-color-text-disabled: #C9CDD4;
  --pf-color-bg-page: #F7F8FA;
  --pf-color-bg-card: #FFFFFF;
  --pf-color-border: #E5E6EB;

  /* 字体 */
  --pf-font-family: "HarmonyOS Sans", "PingFang SC", "Microsoft YaHei", sans-serif;
  --pf-font-size-title: 16px;
  --pf-font-size-subtitle: 14px;
  --pf-font-size-body: 14px;
  --pf-font-size-aux: 12px;

  /* 控件 */
  --pf-control-height: 32px;
  --pf-control-height-sm: 24px;
  --pf-radius: 4px;
  --pf-radius-tag: 2px;

  /* 间距 */
  --pf-spacing-page: 24px;
  --pf-spacing-card: 16px;
  --pf-spacing-field: 16px;

  /* Dialog */
  --pf-dialog-w-tip: 360px;
  --pf-dialog-w-default: 480px;
  --pf-dialog-w-long: 560px;
  --pf-dialog-w-form: 720px;
}
```

---

## 怎么用这份样本

1. 用户给你设计规范截图 / 文档 / Figma 链接
2. 你按上面这个**结构骨架**提取最小集，输出 `prototype/design_tokens.md`
3. 顺手改 `prototype/styles.css` 顶部 `:root` 变量，原型页直接 `var(--pf-color-primary)` 引用
4. 提取过程中看不清的，列在「待补 / 看不清的项」，告诉用户「研发实现时按你们规范覆盖」

> 边界提示：低保真原型只对齐**视觉变量**，不对齐**组件交互细节**（如表单校验时机、Dialog 关闭动画）。这些写在 PRD 正文里就够了。
