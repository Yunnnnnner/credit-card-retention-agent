# 02. 技术栈与环境配置

> 本文件是 PRD/ 文件夹的第 2 部分。如需了解产品全貌请先读 [README.md](./README.md)。
> 上一模块：[01-overview.md](./01-overview.md) · 下一模块：[03-design-tokens.md](./03-design-tokens.md)

---

## 技术栈总览

### 前端原型（已实现）

| 层级 | 技术 | 说明 |
|------|------|------|
| 标记语言 | HTML5 | 单文件结构，2200+ 行 |
| 样式 | CSS3 + CSS Variables | 全局 Design Tokens，深色科技感主题 |
| 脚本 | Vanilla JavaScript | 导航切换、筛选逻辑、数据联动 |
| 图标 | Font Awesome 6.4 (CDN) | 120+ 个图标引用 |
| 字体 | Inter (Google Fonts CDN) | 全局默认字体 |
| 依赖 | 零本地依赖 | 浏览器直接打开即可运行 |

### 后端架构设计（架构总览页展示）

| 层级 | 技术 | 版本 | 用途 |
|------|------|------|------|
| Agent 编排 | LangGraph | 0.2.x | 有向无环图编排四 Agent 流水线 |
| LLM 模型 | DeepSeek-V3 | deepseek-chat | 中文金融场景推理 |
| LLM 接入 | langchain-openai | 0.2.x | 兼容 OpenAI SDK 调用 DeepSeek |
| 数据校验 | Pydantic V2 | 2.x | Agent 输入输出 Schema 强类型约束 |
| RAG 引擎 | LangChain | 0.2.x | 知识库检索 + 上下文注入 |
| 向量数据库 | FAISS | latest | 高性能相似度检索 |
| Embedding | BGE-M3 | — | 多语言嵌入模型 |
| 降级策略 | 规则引擎 | — | Agent 异常时的 fallback |

---

## 当前项目文件结构

```
Desktop/2/
├── index.html                  # 完整前端原型（单文件，可直接浏览器打开）
├── BRD.md                      # 商业需求文档
├── MRD.md                      # 市场需求文档
├── PRD/                        # 产品需求文档（模块化）
│   ├── README.md               # PRD 总览
│   ├── 01-overview.md          # 项目概述
│   ├── 02-tech-stack.md        # 技术栈（本文件）
│   ├── 03-design-tokens.md     # 设计规范
│   ├── 04-pages-components.md  # 页面与组件
│   ├── 05-ai-capabilities.md   # AI 能力定义
│   ├── 06-data-model.md        # 数据模型
│   ├── 07-business-logic.md    # 业务逻辑
│   ├── 08-state-management.md  # 状态管理
│   ├── 09-error-handling.md    # 异常处理
│   ├── 10-roadmap.md           # 路线图
│   └── APPENDIX-data-index.md  # 数据索引
└── market_data/                # 市场调研原始数据
    └── raw_data.md
```

---

## 前端技术细节

### CDN 依赖

```html
<!-- Font Awesome 6.4 图标库 -->
<link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
<!-- Google Fonts Inter -->
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
```

### CSS Variables（核心设计系统）

```css
:root {
    --primary: #6C5CE7;        /* 主题紫 */
    --primary-light: #A29BFE;  /* 浅紫 */
    --success: #00B894;        /* 成功绿 */
    --warning: #FDCB6E;        /* 警告黄 */
    --danger: #E17055;         /* 危险红 */
    --info: #74B9FF;           /* 信息蓝 */
    --bg: #0F0F23;             /* 背景深色 */
    --surface: #1A1A2E;        /* 卡片表面 */
    --border: #2D2D44;         /* 边框色 */
    --text-primary: #E8E8F0;   /* 主文字 */
    --text-secondary: #9090A8; /* 次文字 */
    --text-muted: #5A5A72;     /* 弱文字 */
    --accent-gold: #F4D03F;    /* 金色强调 */
}
```

### JavaScript 核心函数

```javascript
// 页面导航
function showPage(page) { /* 切换 page-section 显隐 + 更新侧边栏 */ }
function showFollowupDetail(cif) { /* 导航到挽回详情页 */ }

// 数据筛选
function filterFollowupList(status) { /* Tab 筛选挽回列表 */ }
function filterCustomersByHeatmap(aum, risk) { /* 热力图 → 客户台账筛选 */ }
function resetCustomerFilter() { /* 清除客户台账筛选 */ }

// KPI 时间切换
function switchTimeRange(range) { /* 切换今日/本周/本月 */ }
```

---

## 后端架构设计详情（架构总览页展示内容）

### Agent 模型参数

| Agent | Temperature | Max Tokens | 场景说明 |
|-------|-------------|------------|---------|
| 客户洞察 | 0.1 | 800 | 低随机性，确保评估一致性 |
| 产品洞察 | 0.3 | 600 | 适度创造性，推荐多样化 |
| 经营策略 | 0.1 | 500 | 低随机性，策略严谨 |
| 营销执行 | 0.5 | 1000 | 较高创造性，话术自然 |

### RAG 知识库检索参数

| 知识库 | 服务 Agent | Top-K | Threshold | 内容 |
|--------|-----------|-------|-----------|------|
| 产品知识库 | 产品洞察 | 5 | 0.75 | 权益目录、费率表、产品规则 |
| 策略知识库 | 经营策略 | 3 | 0.80 | 策略矩阵、历史案例、行业基准 |
| 合规话术库 | 营销执行 | 5 | 0.70 | 话术模板、合规红线、成功案例 |

### 后端对接方案（V2 规划）

```
前端 index.html
  ↕ REST API / WebSocket
后端 Python (FastAPI)
  ├── /api/analyze/{cif}     → 触发 LangGraph 流水线
  ├── /api/batch              → 批量分析
  ├── /api/customers          → 客户数据 CRUD
  └── /api/followup/{cif}     → 挽回记录管理
  ↕
LangGraph Pipeline → DeepSeek-V3 → RAG (FAISS + BGE-M3)
```
