# datasource/crud 目录教程

## 文件

- `datasource.py`：数据源核心服务；获取/保存数据源、测试连接、执行预览、读取表样例、`get_table_schema` 拼装 LLM 上下文。
- `table.py`：表元数据同步、可见性、别名/注释和 embedding 维护。
- `field.py`：字段元数据、类型、注释、排序与可见性维护。
- `permission.py`：数据源/表/字段权限的综合逻辑。
- `row_permission.py`：行级条件与 SQL 过滤规则。
- `recommended_problem.py`：推荐问题和图表预设的持久化。
- `__init__.py`：包声明。

`get_table_schema` 是问数关键：先取得当前用户可见的表字段，再做表 embedding 召回，并把关联表和外键补齐，返回 prompt 字符串与允许表名单。

## 模块协作

`datasource.py` 调用 `apps/db` 读取远端元数据和执行预览；调用 table/field 保存本地治理结果；调用 permission 得到当前用户可见列和行过滤；调用 embedding 选择相关表；关系数据来自 CoreDatasource.table_relation。

## 元数据与实时数据库

系统库保存的是同步快照和人工注释。业务库结构变化后若未同步，模型仍看到旧 Schema，执行时才报字段不存在。运维上需要同步策略和错误反馈闭环，而不是每轮问数都扫描远端数据库。

## 权限生成

列权限应在 Schema 构造阶段隐藏字段；行权限在 SQL 生成后形成 WHERE/过滤 SQL。二者都要按当前 user/oid 计算。管理员和普通用户路径不同，`is_normal_user` 的判断影响 LLMService 是否应用过滤。

## 失败排查

连接错误看解密后参数/driver；无表看 choose flag；Schema 不完整看表 embedding Top-N 和关系补表；权限错误看 DsRules 与变量值；预览正常但问数执行失败看生成 SQL 方言与只读校验。

