# 09. 错误处理与兜底

> 本文件是 PRD/ 文件夹的第 9 部分。如需了解产品全貌请先读 [README.md](./README.md)。
> 上一模块：[08-state-management.md](./08-state-management.md) · 下一模块：[10-roadmap.md](./10-roadmap.md)

---

## 错误分类与处理原则

**原则**：任何单个 Agent 的失败，**不应中断整个流水线**。错误只影响对应 Agent 的输出，后续 Agent 使用降级数据继续执行。

| 错误类型 | 触发场景 | 处理方式 | 用户感知 |
|---------|---------|---------|---------|
| API 超时 | DeepSeek 响应 > 30s | 重试 1 次；失败则使用规则引擎 fallback | 界面显示该 Agent 黄色警告标记 |
| JSON 解析失败 | LLM 返回非 JSON 格式 | 重试 1 次；失败则 fallback | 同上 |
| Pydantic 校验失败 | 输出不符合 Schema | 重试 1 次；失败则 fallback | 同上 |
| 全局流水线崩溃 | 未预期异常（OOM 等）| 捕获顶层 Exception，设置 `pipeline_status="error"` | 红色错误提示 + 重试按钮 |
| 数据加载失败 | CSV 格式错误 / 客户 ID 不存在 | 流水线不启动，Sidebar 显示错误信息 | 红色提示，不显示分析页面 |

---

## 每个 Agent 的超时与重试处理

```python
# 通用 Agent 调用包装器
import time
from pydantic import ValidationError

def run_agent_with_fallback(
    agent_fn: Callable,
    fallback_fn: Callable,
    state: AgentState,
    agent_name: str,
    max_retries: int = 2,
) -> dict:
    """
    调用 agent_fn，失败时最多重试 max_retries 次，
    所有重试失败后调用 fallback_fn 返回降级结果。
    返回值为 AgentState 的部分更新 dict。
    """
    last_error = None
    for attempt in range(max_retries):
        try:
            result = agent_fn(state)
            return result
        except (TimeoutError, ValidationError, ValueError) as e:
            last_error = str(e)
            if attempt < max_retries - 1:
                time.sleep(1)   # 重试间隔 1 秒

    # 所有重试失败，使用降级
    fallback_result = fallback_fn(state)
    return {
        **fallback_result,
        "errors": state.get("errors", []) + [f"{agent_name}: {last_error}"],
        "is_degraded": True,
    }
```

---

## Streamlit 界面错误展示规范

### Loading 状态

每个 Agent 执行时：
```python
with st.spinner(f"⏳ {agent_name} 分析中..."):
    result = run_agent_with_fallback(...)
```

执行完成后立即渲染结果（不等下一个 Agent）。

### 降级警告（黄色）

```python
if is_degraded:
    st.warning("⚠️ AI 服务响应异常，部分结果由规则引擎生成，仅供参考。")
```

### 全局错误（红色）

```python
if st.session_state.pipeline_status == "error":
    st.error("❌ 分析流程出现未预期错误，请重试。")
    if st.button("🔄 重试"):
        st.session_state.pipeline_status = "idle"
        st.rerun()
```

### 空状态

| 场景 | 展示内容 |
|------|---------|
| 未选择客户（页面初始）| 主区域显示：「👈 请在左侧选择一个客户开始分析」 |
| 客户选择后未运行 | 显示客户基础信息卡片，主区域显示四个空白 Agent 卡片（带 `—` 占位） |
| 批量结果为空 | 显示：「暂无批量分析结果，点击「批量运行」开始」|

---

## API Key 缺失处理

```python
# app.py 启动时检查
import os
if not os.getenv("DEEPSEEK_API_KEY"):
    st.error(
        "❌ 未检测到 DEEPSEEK_API_KEY 环境变量。\n\n"
        "请在项目根目录创建 `.env` 文件并填入：\n"
        "```\nDEEPSEEK_API_KEY=sk-your-key-here\n```"
    )
    st.stop()  # 阻止后续渲染
```

---

## pytest 测试覆盖要求

| 测试文件 | 覆盖场景 |
|---------|---------|
| `test_customer_insight.py` | 正常输出、JSON 解析失败降级、超时降级 |
| `test_product_insight.py` | 正常推荐、已持有产品过滤、降级默认推荐 |
| `test_strategy.py` | 全部 9 种策略矩阵组合、边界客户（新客户）|
| `test_marketing.py` | 话术长度校验（<100字重试）、违禁词检测 |
| `test_pipeline.py` | 端到端单客户、全程 LLM 失败时全降级运行 |

> 测试使用 `unittest.mock.patch` 模拟 DeepSeek API，不真实消耗 Token。
