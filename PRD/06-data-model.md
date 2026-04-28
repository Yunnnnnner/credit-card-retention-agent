# 06. 数据模型

> 本文件是 PRD/ 文件夹的第 6 部分。如需了解产品全貌请先读 [README.md](./README.md)。
> 上一模块：[05-ai-capabilities.md](./05-ai-capabilities.md) · 下一模块：[07-business-logic.md](./07-business-logic.md)

> ⚠️ 本项目无真实数据库。所有数据模型为：① Pydantic 模型（Agent I/O 约束）② 模拟 CSV 数据规范（不接真实系统）

---

## 输入模型：Customer（客户数据）

```python
# models/customer.py
from pydantic import BaseModel, Field
from typing import Literal

class Customer(BaseModel):
    # 基础信息
    customer_id: str                        # e.g. "C001"
    age: int = Field(ge=18, le=80)
    gender: Literal["男", "女"]
    months_as_customer: int = Field(ge=1)   # 持卡月数

    # 账户特征
    credit_limit: float = Field(gt=0)       # 信用额度（元）
    current_balance: float = Field(ge=0)    # 当前账单余额（元）
    available_credit: float                 # 可用额度 = credit_limit - current_balance

    # 消费行为（核心流失信号）
    avg_monthly_spend_3m: float = Field(ge=0)   # 近3个月月均消费（元）
    avg_monthly_spend_6m: float = Field(ge=0)   # 前3-6个月月均消费（元）
    days_since_last_transaction: int = Field(ge=0)  # 距上次交易天数

    # 还款记录
    payment_history: Literal["excellent", "good", "fair", "poor"]

    # 产品持有
    product_count: int = Field(ge=1, le=10)     # 持有银行产品数量
    products: list[str]                          # 产品列表，e.g. ["credit_card", "savings"]

    # 渠道偏好
    preferred_channel: Literal["app", "branch", "online", "phone"]

    # 衍生字段（由数据加载时计算，不由 LLM 生成）
    customer_segment: Literal["高价值", "中价值", "低价值"]
    spending_decline_rate: float    # (3m - 6m) / 6m，负值表示下降
```

**客户价值分层规则**（在 `utils/data_loader.py` 中计算，不依赖 LLM）：

| 分层 | 判断条件 |
|------|---------|
| 高价值 | `credit_limit >= 50000` 且 `avg_monthly_spend_3m >= 5000` |
| 中价值 | `credit_limit >= 20000` 且 `avg_monthly_spend_3m >= 2000`（不满足高价值条件）|
| 低价值 | 其他 |

---

## 输出模型汇总

各 Agent 的输出 Pydantic 模型定义在 [05-ai-capabilities.md](./05-ai-capabilities.md)，此处列出类名对照：

```python
# models/outputs.py
from agents.customer_insight import ChurnRiskAssessment
from agents.product_insight   import ProductRecommendation, ProductRecommendationItem
from agents.strategy          import RetentionStrategy
from agents.marketing         import MarketingScript
```

---

## LangGraph 状态模型

```python
# graph/state.py
from typing import TypedDict, Optional
from models.customer import Customer
from models.outputs import (
    ChurnRiskAssessment, ProductRecommendation,
    RetentionStrategy, MarketingScript
)

class AgentState(TypedDict):
    # 输入
    customer_id: str
    customer_data: Customer

    # 各 Agent 输出（顺序填充）
    churn_assessment:      Optional[ChurnRiskAssessment]
    product_recommendation: Optional[ProductRecommendation]
    retention_strategy:    Optional[RetentionStrategy]
    marketing_script:      Optional[MarketingScript]

    # 运行控制
    current_step: str   # "customer_insight" | "product_insight" | "strategy" | "marketing" | "done"
    errors: list[str]   # 每步失败时追加错误信息，不中断流程
```

---

## 模拟数据规范（simulated_customers.csv）

**文件路径**：`data/simulated_customers.csv`
**数据量**：≥ 50 条，覆盖以下分布：

| 客户价值 | 风险等级 | 数量 |
|---------|---------|------|
| 高价值 | 高风险 | 5 条 |
| 高价值 | 中风险 | 5 条 |
| 高价值 | 低风险 | 5 条 |
| 中价值 | 高风险 | 8 条 |
| 中价值 | 中风险 | 8 条 |
| 中价值 | 低风险 | 7 条 |
| 低价值 | 高风险 | 6 条 |
| 低价值 | 中风险 | 6 条 |

**CSV 字段列表**（字段名必须与 `Customer` 模型一致）：

```
customer_id, age, gender, months_as_customer,
credit_limit, current_balance, available_credit,
avg_monthly_spend_3m, avg_monthly_spend_6m,
days_since_last_transaction, payment_history,
product_count, products, preferred_channel,
customer_segment, spending_decline_rate
```

**数据真实性要求**：
- `avg_monthly_spend_3m` 的数值范围参考实际信用卡市场：低价值 500-2000 元，中价值 2000-8000 元，高价值 8000-30000 元
- `days_since_last_transaction`：高风险客户 ≥ 45 天，低风险客户 ≤ 15 天
- `spending_decline_rate`：高风险客户 ≤ -0.3（下降 30% 以上），低风险客户 ≥ -0.1

---

## 产品目录规范（product_catalog.json）

**文件路径**：`data/product_catalog.json`

```json
{
  "products": [
    {
      "id": "P001",
      "name": "积分翻倍礼包",
      "target_segment": ["高价值", "中价值"],
      "selling_point": "消费积分翻倍，全年有效",
      "cost_level": "high",
      "channel": ["app", "phone"]
    },
    {
      "id": "P002",
      "name": "年费减免券",
      "target_segment": ["中价值", "低价值"],
      "selling_point": "下一年度年费全免",
      "cost_level": "medium",
      "channel": ["sms", "app"]
    },
    {
      "id": "P003",
      "name": "专属返现权益（3个月）",
      "target_segment": ["高价值"],
      "selling_point": "指定品类消费返现5%，连续3个月",
      "cost_level": "high",
      "channel": ["phone", "app"]
    },
    {
      "id": "P004",
      "name": "满减优惠券",
      "target_segment": ["中价值", "低价值"],
      "selling_point": "当月消费满500减50",
      "cost_level": "low",
      "channel": ["sms", "app"]
    },
    {
      "id": "P005",
      "name": "机场贵宾厅权益（季度）",
      "target_segment": ["高价值"],
      "selling_point": "国内主要机场贵宾厅季度无限次使用",
      "cost_level": "high",
      "channel": ["phone"]
    }
  ]
}
```
