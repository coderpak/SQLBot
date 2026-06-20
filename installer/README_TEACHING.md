# installer 目录教程

存放服务器安装、控制、配置模板和卸载脚本，不参与本地 FastAPI/Vite 业务运行。

## 文件

- `install.sh`：环境检查、目录准备、配置和容器安装入口。
- `install.conf`：安装器默认参数。
- `sctl`：SQLBot 服务控制命令包装，通常用于 start/stop/status/restart/log 等运维动作。
- `uninstall.sh`：卸载服务；执行前要确认是否保留数据卷。
- `sqlbot/docker-compose.yml`：安装器使用的 Compose 模板。
- `sqlbot/templates/sqlbot.conf`：运行配置模板，由安装脚本替换变量。

## 安装数据流

读取 install.conf/交互参数 → 检查 Docker/端口/权限 → 创建持久化目录 → 渲染 sqlbot.conf 与 compose → 拉取镜像并启动 → 通过 sctl 运维。

## 修改和使用注意

正式部署前修改默认密码和 SECRET_KEY，确认 8000/8001 暴露范围、数据目录容量、备份、日志轮转和镜像版本。卸载脚本最危险：区分“移除容器”和“删除用户数据”，默认不应静默删除数据库和上传文件。

