# 📎 数据索引附录

> 本文件是 PRD/ 文件夹的补充模块。如需了解产品全貌请先读 [README.md](./README.md)。
> 本文件列出 PRD 中引用的所有上游数据索引。每条索引对应 `./market_data/raw_data.md` 的一条原始记录，可追溯验证。

---

## 继承来源

- **BRD.md**：引用索引 = W1, W2, W3, W5, W6, W7, W8, W9, W10, W11, W13, W14, W15, W16, W17, W18, W19, W20
- **MRD.md**：引用索引 = W1, W2, W3, W4, W5, W6, W7, W8, W9, W10, W11, W13, W14, W15, W16, W17, W18, W20
- **原始数据文件**：`./market_data/raw_data.md`（共 21 条原声）

---

## 索引列表（PRD 直接引用）

| 索引 | 来源平台 | 原文摘要 | 链接 | 在 PRD 里支撑了什么 |
|------|---------|---------|------|------------------|
| W5 | 53AI | "流失预警准确率89%，提前15天启动挽留，3个月唤醒23万客户" | https://www.53ai.com/news/zhinengyingxiao/2025051326935.html | [01-overview.md] P0 功能「流失风险识别」的价值依据 |
| W6 | CSDN博客 | "高价值客户流失率降低27%" | https://blog.csdn.net/qq_44654951/article/details/154074607 | [01-overview.md] P0 功能「流失风险识别」的效果背书 |
| W8 | 中关村科金 | "营销转化率提升30%，对话轮次提升84%" | https://www.zkj.com/industry_news/8143.html | [01-overview.md] P0「话术生成」的效果依据；[05-ai-capabilities.md] 营销 Agent System Prompt 话术质量目标 |
| W9 | 53AI | "超过200个触发场景，预测+干预模式" | https://www.53ai.com/news/zhinengyingxiao/2025051326935.html | [07-business-logic.md] 策略矩阵多维触达设计依据 |
| W10 | 腾讯云 | "LangGraph适合金融合规场景，推荐可解释自研方案" | https://cloud.tencent.com/developer/article/2639437 | [02-tech-stack.md] 选 LangGraph 而非 CrewAI 的技术理由 |
| W11 | 新浪新闻 | "2026年LangGraph/CrewAI/AutoGen各有优势" | https://k.sina.cn/article_7857201856_1d45362c00190413au.html | [02-tech-stack.md] 框架选型对比背景 |
| W13 | IT之家 | "LangGraph金融场景领跑，CrewAI快速原型见长" | https://www.ithome.com/0/942/011.htm | [02-tech-stack.md] LangGraph 推荐依据 |
| W16 | 百度AI白皮书 | "差异化产品推荐是城商行核心数字化需求" | https://doc.bce.baidu.com/bce-documentation/analyst-reports/ | [07-business-logic.md] 产品匹配 Agent 设计依据 |
| W18 | Fintech Futures | "2026年Agentic AI从试点转向大规模部署" | https://www.fintechfutures.com/ai-in-fintech/banking-in-2026-production-scale-ai-agents | [10-roadmap.md] V2 多场景扩展的市场时机依据 |
| W20 | AI Journal | "75%大型银行整合AI策略，流失预测成运营标配" | https://aijourn.com/ai-agents-in-financial-services-what-banks-and-fintechs-need-to-know-in-2026/ | [10-roadmap.md] 演示优先级设计依据；[01-overview.md] 技术目标背景 |

---

## 完整索引对应表（来自 raw_data.md）

所有 21 条原始数据均在 `./market_data/raw_data.md` 可查阅，以下为快速索引：

| 索引 | 一句话摘要 |
|------|---------|
| W1 | 2026年银行AI营销市场超2800亿，增速65% |
| W2 | 某银行2000+智能体，客群匹配准确率95% |
| W3 | "帮得助理"让客户经理管理2万+客户 |
| W4 | 开源大模型+智能体平台双引擎降低部署门槛 |
| W5 | 流失预警89%准确，3个月唤醒23万客户 |
| W6 | 交通银行高价值客户流失率降低27% |
| W7 | 获客成本从870元降至356元，降幅59% |
| W8 | 营销大模型转化率+30%，对话轮次+84% |
| W9 | 200+触发场景，预测+干预模式 |
| W10 | LangGraph适合金融合规场景 |
| W11 | LangGraph/CrewAI/AutoGen各有优势 |
| W12 | 微软RD-Agent专注量化金融 |
| W13 | LangGraph金融场景领跑，CrewAI快速原型见长 |
| W14 | 城商行+农商行总资产108.72万亿，转型分水岭 |
| W15 | 中小银行开始集成AI+大数据营销 |
| W16 | 区域银行核心需求：差异化推荐、精准画像 |
| W17 | "7天装上营销大脑"，快速部署成采购核心诉求 |
| W18 | 2026年Agentic AI成银行生存必需 |
| W19 | 大行AI落地：成本降20-40%，收入增10-30% |
| W20 | 75%大型银行整合AI，流失预测成运营标配 |
| W21 | 美国银行业转换率10年高点，Agentic AI趋势 |
