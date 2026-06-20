# template/generate_dynamic 目录教程

- `generator.py`：动态助手/外部数据源场景的 SQL 生成或子查询替换模板。
- `__init__.py`：包声明。

动态助手的数据可能通过外部 API 提供子查询而非 SQLBot 直连。该模板帮助生成带占位子查询的 SQL，`run_task` 在真正执行前把占位替换成外部返回 SQL。替换键、表名白名单和外部 SQL 安全检查是重点。

## 参与者

`AssistantOutDs` 提供外部数据源与 Schema；模板规定动态 SQL 输出；LLMService 保存占位 SQL并请求外部子查询；执行前按表名替换。

## 安全边界

- 只允许替换后端生成的固定前缀占位符。
- 外部返回 SQL 仍视为不可信输入。
- 表名必须位于当前 Schema/允许表列表。
- 外部 endpoint 需要超时、域名约束和响应大小限制。

## 调试

分别记录脱敏后的“模型占位 SQL、外部表到子查询映射、最终 SQL”，才能判断错误出在哪一层。

