# datasource 目录教程

## 作用

管理业务数据源、表、字段、表关系、推荐问题和行列权限；为 AI 问数提供经过授权的 Schema、样例数据与向量召回。

## 子目录

- `api/`：数据源管理接口。
- `crud/`：连接测试、同步元数据、Schema 拼装、权限和推荐问题业务。
- `embedding/`：数据源/表相似度召回。
- `models/`：CoreDatasource/CoreTable/CoreField 等模型和连接 DTO。
- `utils/`：Excel 数据源和配置加解密辅助。
- `__init__.py`：包声明。

## 核心职责不是“存连接字符串”

该模块是 AI 与业务数据库之间的元数据防火墙。它决定模型能看到哪些数据源、表、字段、注释、样例和关系，也决定最终 SQL 应应用哪些行列权限。Text-to-SQL 质量和安全性都高度依赖这里。

## 数据源从创建到问数

1. API 接收连接配置并测试连通性。
2. 敏感配置加密后保存到系统 PostgreSQL。
3. 从业务库读取 schema/table/field，用户选择参与问数的表。
4. 维护自定义表字段注释、字段排序、推荐问题和 X6 关系图。
5. 生成 datasource/table embedding。
6. 问数时按用户权限和问题相似度得到 Schema、样例和允许表名单。
7. 执行前再产生行权限过滤和列权限约束。

## 两种数据源

普通 `CoreDatasource` 由 SQLBot 保存连接并直接访问；动态助手数据源由外部 API 提供，通过 `AssistantOutDs` 适配成类似接口。LLMService 需要同时支持二者，因此不要在 chat 中假设一定能拿到 CoreDatasource ORM 对象。

## 输入与输出

- 管理输入：数据库类型、host/port/database、账号、SSL/超时、选中表、注释、关系和权限。
- AI 输入：自然语言问题、当前用户/工作空间/助手。
- 输出：连接对象、相关 Schema 文本、table_name_list、sample data、过滤 SQL 和统一查询数据。

## 学习实验

选一个有 10 张表的数据源，分别关闭/开启 TABLE_EMBEDDING，打印 `get_table_schema` 返回内容；再修改表注释和关系，观察召回表、生成 SQL 与 token 的变化。这能直观看懂 RAG 的价值。

