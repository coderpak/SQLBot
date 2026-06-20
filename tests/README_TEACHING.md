# tests 目录教程

项目级 pytest 测试目录。后端 `pyproject.toml` 包含 pytest/coverage，`backend/scripts/test.sh` 和 `tests-start.sh` 是容器/CI 入口。

## 现有文件

- `test_cwe89_escape_fix.py`：验证行权限值转义和 SQL 注入修复，属于必须长期保留的安全回归测试。
- `test_minimax_integration.py`：验证 Minimax/OpenAI-compatible 模型集成行为。
- `test_supplier_config.py`：验证模型供应商配置与工厂映射。

## 最值得补的测试

1. `check_sql_read` 对各方言写操作、危险函数、CTE 和多语句的参数化用例。
2. `LLMService.check_sql/check_save_chart` 对夹杂 Markdown、错误 JSON、大小写字段的解析测试。
3. `get_table_schema` 的权限、embedding Top-N 和关系补表测试。
4. `run_task` 的 SSE 事件顺序与 error/finish 行为。
5. 前端 ChartAnswer 的 chunk 拆分、大整数和 AbortController 测试。
6. Alembic 从旧版本数据库升级的集成测试。

测试模型调用时应 mock 或标记为 integration，避免普通单测依赖外网和真实 API Key。

