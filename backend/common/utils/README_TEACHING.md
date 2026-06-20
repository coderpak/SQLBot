# common/utils 目录教程

| 文件 | 作用 |
|---|---|
| `aes_crypto.py` | AES 加解密底层实现。 |
| `crypto.py` | RSA/密钥等通用密码学辅助。 |
| `command_utils.py` | 解析重新生成、分析、预测等聊天快捷命令。 |
| `data_format.py` | 大整数、Decimal、字段 key、Pandas/Excel 数据格式转换。 |
| `embedding_threads.py` | 启动时补齐术语、训练样例、表和数据源 embedding。 |
| `excel.py` | 通用 Excel 读写与格式辅助。 |
| `http_utils.py` | 外部 HTTP 请求和错误处理辅助。 |
| `locale.py` | 后端语言识别与 i18n。 |
| `random.py` | 随机字符串/凭证辅助。 |
| `snowflake.py` | 分布式 Snowflake ID 生成器。 |
| `time.py` | 时间戳与日期转换。 |
| `tree_utils.py` | 权限树/资源树组装。 |
| `utils.py` | 杂项工具、JSON 块提取、hash、比较和 `SQLBotLogUtil`。 |
| `whitelist.py` | 路径或访问白名单规则。 |
| `__init__.py` | 包声明。 |

问数链路重点读 `command_utils.py`、`data_format.py`、`embedding_threads.py`、`utils.py::extract_nested_json`。

## 数据格式为什么重要

数据库可能返回 Decimal、datetime、LOB、超长整数和带表前缀的列名。`data_format.py` 在 API、Pandas、Excel 和图表之间做统一转换。超过 JavaScript 安全整数范围的值必须转字符串；字段规范化要与 chart axis 的小写策略一致。

## Embedding 补偿任务

启动时 `embedding_threads.py` 扫描缺少向量的术语、SQL 样例、表和数据源，再调用各模块的 save_embeddings。它是数据迁移/升级后的补偿机制，不等于实时更新；创建和编辑记录仍应主动保存新向量。

## 模型输出解析

模型常在 JSON 前后附加解释或 Markdown code fence。`extract_nested_json` 用栈找到第一个完整且可解析的对象/数组。它只能解决“夹杂文字”，不能替代 chart schema 校验；拿到 JSON 后仍需验证 type、axis 和字段。

## 工具代码治理

只有真正跨领域、无业务状态的逻辑才放这里。涉及用户权限、数据源、ChatRecord 的函数应留在对应模块。改动 snowflake、加密、data_format 或日志工具前先查调用者，因为影响面远大于单个业务文件。

