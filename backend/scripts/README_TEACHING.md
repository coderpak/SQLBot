# scripts 目录教程

| 文件/目录 | 作用 |
|---|---|
| `prestart.sh` | 容器启动前准备，通常执行数据库可用性等待和迁移。 |
| `test.sh` | pytest/coverage 测试入口。 |
| `tests-start.sh` | CI/容器中的测试前置与启动包装。 |
| `lint.sh` | ruff/mypy 等静态检查入口。 |
| `format.sh` | 自动格式化入口。 |
| `alembic/` | 手工执行或生成迁移的脚本。 |

这些是 Shell 脚本，Windows 本地可在 WSL/Git Bash 中运行，或直接执行其内部对应的 uv 命令。

## 使用边界

prestart 面向服务启动前的确定性准备，不应运行无限时后台任务；test/lint/format 应在 CI 和本地给出一致结果；脚本需 `set -e`/错误传播，避免迁移失败后仍启动应用。生产执行前确认工作目录、环境变量和 Python 虚拟环境。

