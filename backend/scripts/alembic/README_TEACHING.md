# scripts/alembic 目录教程

- `auto.sh`：根据 SQLModel metadata 与当前数据库差异自动生成迁移草稿；生成后必须人工审查。
- `exec.sh`：执行 Alembic upgrade 等迁移命令的包装脚本。

不要把 autogenerate 当成数据库设计工具；重命名、数据回填和兼容升级通常需要手写迁移。

## `auto.sh`

通常调用 Alembic revision --autogenerate 并接收迁移说明。运行前要连接开发数据库且其 revision 与代码基线一致；生成文件后检查 down_revision 是否正确。

## `exec.sh`

封装 upgrade/downgrade/current/history 等执行。生产执行前备份数据库，记录当前 revision，并确保同一时刻只有一个实例做迁移。

## CI 建议

在空库执行 upgrade head，再从一个旧版本 fixture 升级；检查 heads 只有一个，避免多人并行迁移形成未合并分支。

