# installer/sqlbot 部署模板模块

- `docker-compose.yml`：安装器最终部署 SQLBot 容器、端口、环境变量、网络与持久化卷的模板。
- `templates/`：由安装脚本填充的运行配置。

安装脚本通常把本目录复制/渲染到目标安装路径，再调用 Docker Compose。修改服务名、卷或端口时要同步 `sctl` 的容器查找和健康检查逻辑。

生产配置要固定镜像版本而非漂移 latest，并显式持久化 PostgreSQL、上传文件、Excel、图片和日志目录。

