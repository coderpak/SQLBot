# apps/template 目录教程

## 作用

提供 Python 侧提示词加载入口。真正的长文本在 `backend/templates/template.yaml` 与 `sql_examples/*.yaml`。

## 文件和子目录

- `template.py`：缓存读取 YAML；按 `DB` 枚举选择方言模板，支持清缓存重载。
- `generate_sql/`：SQL prompt 入口。
- `generate_chart/`：图表 prompt 入口，也暴露术语/训练基本模板。
- `generate_analysis/`：智能分析 prompt。
- `generate_predict/`：预测 prompt。
- `generate_guess_question/`：推荐追问 prompt。
- `select_datasource/`：自动选择数据源 prompt。
- `filter/`：权限 SQL/过滤条件 prompt。
- `generate_dynamic/`：动态数据源 SQL prompt。
- `__init__.py`：包声明。

这里的 generator 很薄，核心价值是把业务代码与 YAML 键名解耦。

## Prompt 是内部协议

SQLBot 依赖模型返回可解析 SQL 和图表 JSON，因此模板不仅是“文案”。`LLMService.check_sql`、`check_save_chart`、前端 Chart 类型都与模板输出约定绑定。修改键名、标记或 JSON 结构时，要像改 API 一样做兼容设计。

## SQL 模板层次

通用模板规定角色、安全、输出结构、Schema/术语/样例插槽和行数限制；数据库 YAML 规定引用符、LIMIT/TOP/FETCH、函数、日期和基础示例；`chat_model.py` 在运行时填入当前 engine、Schema、sample data、历史和问题。

## 缓存

`template.py::_load_template_file` 使用 `@cache`。生产中能减少每轮磁盘 IO；开发时修改 YAML 后需要重启服务或显式 `reload_all_templates`，否则看似修改成功但实际仍使用旧内容。

## 调试方法

从 ChatLog 查看最终完整 messages，而不是只看 YAML 某一段。模型行为可能同时受系统规则、方言示例、术语、训练 SQL、历史和自定义提示词影响。

