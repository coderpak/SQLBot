# settings 目录教程

## 作用

系统设置业务域，管理可在 UI 中维护的基础配置。

- `api/`：设置接口。
- `models/`：设置持久化模型。
- `schemas/`：请求/响应结构。
- `__init__.py`：包声明。

## 当前边界

这个模块名称很宽，但当前开源代码中的公开能力较薄：`api/base.py` 主要根据 `FileRequest` 从 Excel 文件目录提供下载。真正的系统参数 CRUD 位于 `apps/system/api/parameter.py` 与 `crud/parameter_manage.py`，启动环境则由 `common/core/config.py` 读取 `.env`。

理解这一边界很重要：不要假设 UI 中所有“系统设置”都从本模块读取，也不要把需要重启的基础设施配置做成普通在线参数。

文件下载要防止路径穿越，只允许受控目录和合法文件标识；响应应使用流式/文件响应而不是一次性读入大文件。

