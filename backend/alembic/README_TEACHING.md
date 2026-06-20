# alembic 目录教程

## 文件

- `env.py`：读取 SQLBot 系统库 URL 和 SQLModel metadata，配置在线/离线迁移。
- `script.py.mako`：新迁移文件模板。
- `README`：Alembic 上游说明。
- `versions/`：按版本顺序保存系统库结构变更。

`main.py::lifespan` 每次启动自动 `upgrade head`。迁移必须可在已有生产数据上执行，不能只验证空库。

## 迁移开发流程

先改 SQLModel → 生成 revision 草稿 → 人工检查 upgrade/downgrade、约束名、默认值和数据回填 → 在旧版本备份数据库执行 upgrade → 启动新代码验证 → 必要时测试 downgrade。自动生成无法可靠识别字段重命名，会误判为删列+加列。

## pgvector

术语迁移会执行 `CREATE EXTENSION IF NOT EXISTS vector`。数据库实例必须安装 pgvector，首次迁移账号要有创建扩展权限。应用账号可在扩展创建后收紧权限。

