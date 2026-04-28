# 05. AI 能力配置

> 本文件是 PRD/ 文件夹的第 5 部分。如需了解产品全貌请先读 [README.md](./README.md)。
> 上一模块：[04-pages-components.md](./04-pages-components.md) · 下一模块：[06-data-model.md](./06-data-model.md)

---

## LangGraph 图结构

```python
# graph/pipeline.py 核心结构

from langgraph.graph import StateGraph, END
from graph.state import AgentState

def build_pipeline() -> CompiledGraph:
    graph = StateGraph(AgentState)

    # 注册四个节点
    graph.add_node("customer_insight",  customer_insight_node)
    graph.add_node("product_insight",   product_insight_node)
    graph.add_node("strategy",          strategy_node)
    graph.add_node("marketing",         marketing_node)

    # 线性流水线：无条件边
    graph.set_entry_point("customer_insight")
    graph.add_edge("customer_insight", "product_insight")
    graph.add_edge("product_insight",  "strategy")
    graph.add_edge("strategy",         "marketing")
    graph.add_edge("marketing",        END)

    return graph.compile()
```

> V1 使用线性图（无条件分支），简单可控。V2 可在高风险路径加条件边（例如高风险客户跳过产品洞察直接进入策略节点）。

---

## Agent 1：客户洞察 Agent

**文件**：`agents/customer_insight.py`

### System Prompt

```
你是一名资深银行信用卡运营分析师，拥有10年信用卡客户流失分析经验。

你的任务是根据客户的消费行为和账户特征，判断该客户的流失风险，并给出3条具体的风险依据。

【评估维度】
1. 消费趋势：近3个月月均消费 vs 前3个月月均消费，下降幅度
2. 活跃度：距上次交易天数（>60天为深度沉睡，30-60天为轻度沉睡）
3. 还款记录：poor/fair 还款记录是流失高风险信号
4. 产品绑定：持有产品数越少（=1），流失风险越高
5. 客户价值：高价值客户（credit_limit > 50000）流失影响更大，需优先标注

【输出要求】
- 风险等级：高风险 / 中风险 / 低风险（三选一）
- 风险评分：0-100的整数（70+为高风险，40-69为中风险，<40为低风险）
- 3条风险依据：每条依据必须引用具体数据（如"近3个月月均消费下降42%"）
- 客户价值分层：高价值 / 中价值 / 低价值
- 一句话总结：对该客户流失风险的简洁总结，不超过50字

严格按照 JSON 格式输出，不要有任何多余文字。
```

### 输出 Schema（Pydantic）

```python
class ChurnRiskAssessment(BaseModel):
    risk_level: Literal["高风险", "中风险", "低风险"]
    risk_score: int = Field(ge=0, le=100)
    risk_reasons: list[str] = Field(min_length=3, max_length=3)
    customer_value: Literal["高价值", "中价值", "低价值"]
    summary: str = Field(max_length=50)
```

### 降级策略

| 场景 | 处理方式 |
|------|---------|
| API 超时（> 30s） | 重试 1 次；仍失败则返回 `risk_level="中风险", risk_score=50`，标注 `error="timeout"` |
| JSON 解析失败 | 用 `model_validate_json()` 捕获 `ValidationError`，重试 1 次；仍失败则降级 |
| 输出不符合 Schema | Pydantic 自动校验，失败则降级到规则引擎：消费下降>30% → 高风险，否则中风险 |

---

## Agent 2：产品洞察 Agent

**文件**：`agents/product_insight.py`

### System Prompt

```
你是一名银行信用卡产品专家，熟悉各类信用卡权益和挽留产品。

你的任务是根据客户画像和流失风险评级，从产品目录中匹配1-3个最适合挽留该客户的产品或权益，并说明匹配理由。

【匹配原则】
- 高价值客户：优先匹配高端权益（专属积分翻倍、机场贵宾厅、专属客户经理）
- 中价值客户：匹配实用权益（返现、年费减免、消费满减）
- 低价值客户：匹配低成本权益（积分奖励、便民权益）
- 已持有的产品不要重复推荐
- 根据客户消费偏好（餐饮/出行/购物）选择对应权益

【产品目录】
参见 data/product_catalog.json

【输出要求】
- 推荐1-3个产品/权益
- 每个推荐必须包含：产品名称、核心卖点（1句话）、匹配理由（引用客户具体特征）

严格按照 JSON 格式输出，不要有任何多余文字。
```

### 输出 Schema

```python
class ProductRecommendationItem(BaseModel):
    product_name: str
    selling_point: str = Field(max_length=50)
    match_reason: str = Field(max_length=100)

class ProductRecommendation(BaseModel):
    recommendations: list[ProductRecommendationItem] = Field(min_length=1, max_length=3)
    recommendation_summary: str = Field(max_length=80)
```

### 降级策略

| 场景 | 处理方式 |
|------|---------|
| API 超时 | 重试 1 次；仍失败则按客户价值返回默认推荐：高价值→积分翻倍，中→年费减免，低→积分奖励 |
| 推荐产品不在目录中 | 与 `product_catalog.json` 比对，不匹配则替换为同类目中评分最高的产品 |

---

## Agent 3：经营策略 Agent

**文件**：`agents/strategy.py`

### System Prompt

```
你是一名银行零售客户经营专家，负责制定差异化客户挽回策略。

你的任务是根据客户的流失风险评级、价值分层和推荐产品，制定一套完整的挽回策略，包括触达渠道、触达时机、优先级和跟进建议。

【策略矩阵（必须遵守）】
高价值 + 高风险 → 触达方式：专属客户经理电话；时机：立即（24小时内）；优先级：P0
高价值 + 中风险 → 触达方式：App push + 短信；时机：3天内；优先级：P1
中价值 + 高风险 → 触达方式：外呼 + 短信；时机：48小时内；优先级：P1
中价值 + 中风险 → 触达方式：短信 + App push；时机：7天内；优先级：P2
低价值 + 任意  → 触达方式：短信；时机：7天内；优先级：P3
低风险（任意价值）→ 触达方式：定期短信关怀；时机：下个账单日前；优先级：P3

【输出要求】
- 触达渠道（从上面的策略矩阵中选）
- 触达时机
- 策略优先级（P0/P1/P2/P3）
- 核心挽回动作（1-2句话）
- 跟进建议（如首次触达无响应，7天后的跟进方式）

严格按照 JSON 格式输出，不要有任何多余文字。
```

### 输出 Schema

```python
class RetentionStrategy(BaseModel):
    contact_channel: str
    contact_timing: str
    priority: Literal["P0", "P1", "P2", "P3"]
    core_action: str = Field(max_length=100)
    followup_suggestion: str = Field(max_length=100)
    strategy_type: str  # 用于批量分析汇总，e.g. "专属客户经理-立即"
```

### 降级策略

| 场景 | 处理方式 |
|------|---------|
| API 超时 | 重试 1 次；仍失败则根据策略矩阵规则引擎直接计算，不调用 LLM |
| 优先级不在枚举范围 | Pydantic 校验捕获，强制映射到最近的合法值 |

---

## Agent 4：营销执行 Agent

**文件**：`agents/marketing.py`

### System Prompt

```
你是一名银行营销文案专家，擅长写个性化、合规的信用卡挽留话术。

你的任务是根据客户信息、挽回策略和推荐产品，生成一段个性化营销话术，适用于外呼开场白或短信内容。

【话术要求】
1. 开头点出客户关心的点（如账户权益到期、专属优惠）
2. 中间介绍推荐产品/权益的核心价值，数字具体化
3. 结尾给出明确的行动号召（如"点击激活"、"回复1领取"）
4. 语气：亲切专业，不能过于推销，避免"最优惠"、"绝对"等夸张表述
5. 长度：100-200字（外呼开场白或短信正文）
6. 合规要求：
   - 不承诺具体金额或收益率
   - 不使用"免费""赠送"等词（改用"赠送"→"专属权益"）
   - 末尾加：「如需退订请回复TD」（仅短信）

【输出要求】
- 话术类型（外呼开场白 / 短信正文，根据策略中的触达渠道判断）
- 话术正文（100-200字）
- 关键卖点（1句话提炼）

严格按照 JSON 格式输出，不要有任何多余文字。
```

### 输出 Schema

```python
class MarketingScript(BaseModel):
    script_type: Literal["外呼开场白", "短信正文"]
    script_content: str = Field(min_length=100, max_length=200)
    key_selling_point: str = Field(max_length=50)
```

### 降级策略

| 场景 | 处理方式 |
|------|---------|
| API 超时 | 重试 1 次；仍失败则使用模板话术（按客户价值分层的预设模板） |
| 话术长度不符（< 100 或 > 200 字） | Pydantic 校验捕获 `ValidationError`，重试 1 次 |
| 含违禁词（免费、最优惠等） | 前端展示时标黄提示，不阻断流程 |

---

## DeepSeek 调用通用配置

```python
# 所有 Agent 共用的调用模式
from langchain_core.messages import HumanMessage, SystemMessage
from utils.llm_client import get_llm

def call_agent_llm(system_prompt: str, user_content: str) -> str:
    llm = get_llm()
    messages = [
        SystemMessage(content=system_prompt),
        HumanMessage(content=user_content),
    ]
    response = llm.invoke(messages)
    return response.content  # 字符串，再用 Pydantic 解析
```

所有 LLM 调用参数详见 [02-tech-stack.md](./02-tech-stack.md) 的 `utils/llm_client.py`。
