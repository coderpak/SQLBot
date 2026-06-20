# template/select_datasource 目录教程

- `generator.py`：没有预先选择数据源时，生成“从候选数据源中选择最相关项”的 prompt。
- `__init__.py`：包声明。

输入授权后的候选数据源摘要与问题，输出一个数据源 ID/选择理由。候选列表必须先在后端按工作空间过滤；模型只能在列表中选择，返回未知 ID 要拒绝。数据源 embedding 可先缩小候选，LLM 再做语义判断。

## 触发条件

Chat 尚未绑定 datasource，而请求允许自动选择时触发。已有数据源则走 `validate_history_ds`，避免同一会话中无意切库。

## 处理步骤

获取授权候选 → 可选 embedding 预筛 → 格式化候选摘要 → LLM 选择 → `extract_nested_json` 解析 → 校验 ID 确实在候选 → 保存到 ChatRecord/Chat → 发 datasource SSE。

## 失败策略

没有候选、模型选未知 ID 或连接失败都应返回明确错误，不要默认选择第一个可能包含敏感数据的数据源。

