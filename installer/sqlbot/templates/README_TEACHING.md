# installer/sqlbot/templates 配置模板教程

- `sqlbot.conf`：安装时生成的环境变量文件模板，包含数据库、端口、路径、密钥、镜像或服务参数占位符。

模板变量由 `install.sh`/install.conf 替换。新增变量时要同步三处：模板占位符、安装脚本默认/校验、Docker Compose 引用。密码和 SECRET_KEY 不应使用仓库固定默认值；生成后的配置文件权限应仅允许管理员读取。

