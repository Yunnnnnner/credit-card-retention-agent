# 03. Design Tokens（Streamlit 主题配置）

> 本文件是 PRD/ 文件夹的第 3 部分。如需了解产品全貌请先读 [README.md](./README.md)。
> 上一模块：[02-tech-stack.md](./02-tech-stack.md) · 下一模块：[04-pages-components.md](./04-pages-components.md)

---

> ⚠️ 本项目为 Python/Streamlit Demo，无传统前端框架。设计 Token 通过 `.streamlit/config.toml` 实现，辅以少量内联 CSS。

---

## 色彩系统

| 用途 | Token 名 | Hex 值 | 说明 |
|------|---------|--------|------|
| 主色（品牌蓝） | `primary` | `#1E3A5F` | 银行风格深海军蓝，导航栏、按钮主色 |
| 强调色（金色） | `accent` | `#C9A84C` | 金融品牌金，风险标签、高价值客户标识 |
| 背景色 | `background` | `#F5F7FA` | 页面底色，浅灰白 |
| 卡片背景 | `surface` | `#FFFFFF` | Agent 输出卡片背景 |
| 主文字 | `text_primary` | `#1A1A2E` | 正文深色 |
| 次要文字 | `text_secondary` | `#64748B` | 标签、描述性文字 |
| 成功/低风险 | `success` | `#16A34A` | 低流失风险标签 |
| 警告/中风险 | `warning` | `#D97706` | 中流失风险标签 |
| 危险/高风险 | `danger` | `#DC2626` | 高流失风险标签 |
| 边框 | `border` | `#E2E8F0` | 卡片边框、分割线 |

---

## Streamlit 主题配置（.streamlit/config.toml）

```toml
[theme]
primaryColor = "#1E3A5F"
backgroundColor = "#F5F7FA"
secondaryBackgroundColor = "#FFFFFF"
textColor = "#1A1A2E"
font = "sans serif"
```

---

## 补充 CSS（在 app.py 中通过 st.markdown 注入）

```python
CUSTOM_CSS = """
<style>
/* Agent 输出卡片 */
.agent-card {
    background: #FFFFFF;
    border: 1px solid #E2E8F0;
    border-radius: 8px;
    padding: 16px 20px;
    margin-bottom: 12px;
}

/* 风险标签 */
.risk-high   { color: #DC2626; font-weight: 600; }
.risk-medium { color: #D97706; font-weight: 600; }
.risk-low    { color: #16A34A; font-weight: 600; }

/* Agent 步骤标题 */
.agent-header {
    font-size: 14px;
    font-weight: 700;
    color: #1E3A5F;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    margin-bottom: 8px;
    padding-bottom: 8px;
    border-bottom: 2px solid #1E3A5F;
}

/* 推理链路说明文字 */
.reasoning-text {
    font-size: 13px;
    color: #64748B;
    font-style: italic;
    margin-top: 4px;
}
</style>
"""
```

---

## 间距系统（基于 8px 栅格）

| 用途 | 值 |
|------|---|
| 组件内边距（小）| 8px |
| 组件内边距（标准）| 16px |
| 卡片内边距 | 20px |
| 组件间距 | 12px |
| 区块间距 | 24px |
| 页面左右边距 | Streamlit 默认（约 2rem） |

---

## MVP 涉及组件样式指引

| 组件 | 样式要求 |
|------|---------|
| 客户选择下拉框 | Streamlit `st.selectbox`，标签"选择客户"，宽度 100% |
| 「开始分析」按钮 | `st.button`，type="primary"，全宽 |
| Agent 输出卡片 | 用 `st.container` + `st.markdown` + `.agent-card` CSS 类实现 |
| 风险标签 | 根据风险等级注入 `.risk-high / .risk-medium / .risk-low` CSS 类 |
| 进度指示器 | `st.progress` + `st.spinner`，每个 Agent 执行时显示 |
| 批量结果表格 | `st.dataframe`，宽度 100%，支持排序 |
| 导出按钮 | `st.download_button`，导出 CSV |
