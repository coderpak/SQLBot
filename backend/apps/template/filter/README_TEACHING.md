# template/filter 目录教程

- `generator.py`：从基础 YAML 读取权限过滤/SQL 改写提示词。
- `__init__.py`：包声明。

输出格式必须与 `LLMService.generate_filter` 的解析逻辑一致。

输入通常是原 SQL、表结构和当前用户的权限规则，输出是加入过滤条件的安全 SQL/结构化结果。它只负责读取模板，权限实体计算仍在 datasource，模型调用与保存仍在 LLMService。测试时要覆盖已有 WHERE、JOIN、GROUP BY、子查询和变量值。

## 输入来源

行权限条件由 `datasource/crud/permission.py` 和 `row_permission.py` 计算；原 SQL 已经过初步解析；表列表来自允许 Schema。

## 输出处理

模型/规则生成的过滤 SQL还要进入 `check_save_sql` 和最终 `exec_sql` 只读校验。权限 SQL不能因为解析失败就退回无权限的原 SQL；安全默认应是拒绝请求。

## 重点测试

AND/OR 优先级、原 SQL 已有 WHERE、LEFT JOIN 语义、聚合前后过滤、引号转义、系统变量空值、多表同名字段。

