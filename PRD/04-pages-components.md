# 04. 页面与组件清单

> 本文件是 PRD/ 文件夹的第 4 部分。如需了解产品全貌请先读 [README.md](./README.md)。
> 上一模块：[03-design-tokens.md](./03-design-tokens.md) · 下一模块：[05-ai-capabilities.md](./05-ai-capabilities.md)

---

## 页面结构总览

本项目为单文件 HTML 应用（`index.html`），通过 `showPage(page)` JS 函数切换 9 个 `div.page-section` 的显隐。

```
index.html
├── <nav.top-nav> — 顶部导航栏
│   ├── .logo-icon (fa-credit-card) + 系统标题
│   ├── 系统状态指示灯 (Agent / RAG / 向量库)
│   └── 用户头像 + 角色信息
│
├── <div.search-container> — 全局搜索框（CIF/姓名/证件号）
│
├── <aside.sidebar> — 左侧导航栏
│   ├── 核心功能组：运营看板 / 预警中心 / 智能分析 / 挽回跟踪
│   ├── 运营工具组：批量任务 / 数据导入
│   └── 系统管理组：客户台账 / 架构总览
│
└── <main> — 主内容区（9 个 page-section，同一时间只显示 1 个）
    ├── #page-dashboard
    ├── #page-alerts
    ├── #page-analysis
    ├── #page-followup-detail
    ├── #page-followup
    ├── #page-batch
    ├── #page-import
    ├── #page-customers
    └── #page-flow
```

---

## 各页面 DOM 结构

### M1: 运营看板 (`#page-dashboard`)

```
page-header（标题 + 时间选择按钮组 + 导出按钮）
├── KPI 卡片行（5 列 grid）
│   ├── 流失预警客户 1,247（含 ↑12%）
│   ├── A级紧急 326（含 ↑8%）
│   ├── 已挽回 489（含 ↑24%）
│   ├── Agent任务 3,842（含 ↑156%）
│   └── AUM保全 ¥4,280万（含 ↑18%）
├── 双列布局
│   ├── 左：风险热力图 heatmap-table（3×3 矩阵，单元格可点击）
│   │   └── onclick="filterCustomersByHeatmap('钻石','A')" 等
│   └── 右：策略执行环形图 + 多维分析柱状图
└── 多维分析卡片（沉默天数分布 / 消费下降率 / MGM渠道偏好）
```

**交互**：
- 时间按钮组：今日/本周/本月（JS `switchTimeRange()`）
- 热力图单元格点击 → `filterCustomersByHeatmap(aum, risk)` → 跳转 customers 页并筛选

### M2: 预警中心 (`#page-alerts`)

```
page-header（标题 + 批量触发按钮）
├── SLA 统计卡片行（4 列 grid）
│   ├── P0 即时响应 (8)
│   ├── P1 48h响应 (15)
│   ├── P2 7天跟进 (24)
│   └── 今日已处理 (23)
├── Tab 筛选（全部47 / P0(8) / P1(15) / P2(24)）
└── 预警列表（7 条 alert-item）
    └── 每条含：客户名、CIF、AUM tag、风险评分、沉默天数、消费下降率、剩余SLA
    └── onclick="showPage('analysis')"
```

### M3: 智能分析 (`#page-analysis`)

```
page-header（标题 + Agent 按钮组 + 导出报告按钮）
├── 客户画像卡片
│   └── 7 维度：授信额度 / 近3月月均 / 前3月月均 / 沉默天数 / 持卡龄 / MGM偏好 / 产品数
├── Agent 卡片列表（4 个 agent-card）
│   ├── 客户洞察 Agent（3.2s）→ 风险评级A级78分 + 3条依据 + 推理链路
│   ├── 产品洞察 Agent（2.8s）→ 3项权益推荐（返现/贵宾厅/积分翻倍）
│   ├── 经营策略 Agent（2.5s）→ 触达渠道 + SLA + 策略分型 + 核心动作
│   └── 营销执行 Agent（3.8s）→ 外呼话术158字 + 合规检查4项
└── 开始挽回操作按钮
    └── onclick="showPage('followup-detail')"
```

### M4: 挽回操作详情 (`#page-followup-detail`)

```
page-header（标题 + 3 个导航按钮）
│   ├── 返回跟踪列表 → showPage('followup')
│   ├── 查看分析 → showPage('analysis')
│   └── 记录联络（锚点跳转）
├── 客户信息卡（姓名/CIF/AUM/风险/SLA/授信/沉默天数）
├── SOP 流程步骤（5 步）
│   ├── Step 1: 确认客户信息
│   ├── Step 2: 情感关怀
│   ├── Step 3: 权益告知
│   ├── Step 4: 消费引导
│   └── Step 5: 后续跟进
├── 操作清单（6 个 checkbox）
├── 联络时间线（3 条历史记录）
├── 联络记录表单
│   ├── 联络方式 select（电话/App/短信/网点）
│   ├── 联络结果 select（接通-有意向/接通-考虑/未接通/拒绝）
│   ├── 备注 textarea
│   └── 提交按钮
├── 数据回流管道图（5 步：CRM→ETL→模型→策略→看板）
└── 效果评估指标（4 维度：消费恢复/权益使用/二次沉默/NPS）
```

### M5: 挽回跟踪列表 (`#page-followup`)

```
page-header（标题 + 筛选/导出按钮）
├── KPI 汇总卡片行（5 列 grid，可点击筛选）
│   ├── 挽回总量 24（onclick="filterFollowupList('all')"）
│   ├── 挽回成功 14 / 58.3%
│   ├── 跟进中 7
│   ├── 挽回失败 3
│   └── AUM保全 ¥4.2M
├── 效果图表行（3 列 grid）
│   ├── 挽回结果分布环形图
│   ├── SLA达标率进度条（P0:100% / P1:94% / P2:88% / 综合:96%）
│   └── 核心效果指标（消费恢复64.2% / AUM保全91.2% / 权益使用47.8% / 二次沉默11.3% / NPS 34）
└── 客户明细 card
    ├── card-header（标题 + Tab筛选：全部/成功/跟进中/失败）
    └── data-table #followupTable（24 行）
        └── 每行含：CIF/姓名/AUM/风险/评分/状态/SLA/首联/联络次数/恢复率/详情链接
        └── 详情 onclick="showFollowupDetail('CIF-xxx')"
```

**交互**：
- Tab 筛选 → `filterFollowupList(status)` → 按 `data-status` 过滤行 + 更新标题计数
- KPI 卡片点击 → 同 Tab 筛选效果

### M6: 批量任务 (`#page-batch`)

```
page-header（标题 + 导出CSV/执行按钮）
└── card
    ├── card-header（批量分析结果·8条 + Agent成功率95%）
    └── data-table（8 行，含 Agent 各步骤状态 badge）
```

### M7: 数据导入 (`#page-import`)

```
page-header（标题 + 模板下载/开始导入按钮）
├── CSV 模板规范 card（字段说明表格）
├── 拖拽上传区域
└── 校验规则说明
```

### M8: 客户台账 (`#page-customers`)

```
page-header
├── #customerPageTitle + #customerPageSub（动态标题，筛选时变化）
├── #customerResetBtn（清除筛选按钮，初始隐藏）
├── 高级筛选 / 导出按钮
└── data-table #customerTable（8 行）
    └── 每行含 data-aum="钻石" data-risk="A" 属性
    └── 操作列：分析 → onclick="showPage('analysis')"
```

**交互**：
- 从热力图跳转时自动按 AUM+风险筛选
- 点击「清除筛选」→ `resetCustomerFilter()` → 恢复全量

### M9: 架构总览 (`#page-flow`)

```
page-header（标题 + 技术文档/API Spec 按钮）
├── 业务流程图（数据源→RAG→Agent→输出，含连接箭头）
├── Agent 边界与能力表（4 行，含输入/输出/能力/限制/RAG）
├── 核心 Prompt 展示（4 个 code block）
├── Pydantic Schema 展示（4 个 code block）
├── 技术架构参数表（9 项）
└── 策略矩阵表（AUM×风险 → 渠道+SLA+时效）
```

---

## JavaScript 函数清单

| 函数 | 参数 | 作用 |
|------|------|------|
| `showPage(page)` | `string` | 切换 page-section 显隐 + 更新侧边栏 active 状态 |
| `showFollowupDetail(cif)` | `string` | 导航到挽回详情页（对应 CIF） |
| `filterFollowupList(status)` | `'all'\|'success'\|'progress'\|'fail'` | 按状态筛选挽回跟踪表 + 更新 Tab active + 标题计数 |
| `filterCustomersByHeatmap(aum, risk)` | `string, string` | 跳转客户台账并按 AUM+风险筛选 + 更新标题 |
| `resetCustomerFilter()` | — | 清除客户台账筛选，恢复全量显示 |
| `switchTimeRange(range)` | `'today'\|'week'\|'month'` | 切换看板 KPI 时间范围（按钮 active 状态切换） |

---

## 侧边栏导航映射

```javascript
const sidebarMap = {
    dashboard: 0,
    alerts: 1,
    analysis: 2,
    followup: 3,
    'followup-detail': 3,  // 详情页共享挽回跟踪的侧边栏高亮
    batch: 4,
    import: 6,
    customers: 7,
    flow: 9
};
```

---

## 用户操作 → 系统响应

| 用户操作 | 系统响应 |
|---------|---------|
| 点击侧边栏菜单项 | `showPage(page)` → 切换页面 + 更新侧边栏高亮 |
| 点击热力图单元格 | `filterCustomersByHeatmap()` → 跳转 customers + 筛选 + 更新标题 |
| 点击清除筛选 | `resetCustomerFilter()` → 恢复全量 + 隐藏清除按钮 |
| 点击预警项 | `showPage('analysis')` → 跳转智能分析 |
| 点击「开始挽回操作」 | `showPage('followup-detail')` → 跳转挽回详情 |
| 点击挽回跟踪「详情」 | `showFollowupDetail(cif)` → 跳转挽回详情 |
| 点击挽回详情「返回列表」 | `showPage('followup')` → 返回跟踪列表 |
| 点击 Tab 筛选 | `filterFollowupList(status)` → 过滤行 + 更新计数 |
| 点击 KPI 卡片 | 对应筛选函数 → 同 Tab 筛选效果 |
| 点击客户台账「分析」 | `showPage('analysis')` → 跳转智能分析 |
