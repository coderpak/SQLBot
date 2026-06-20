# settings/api 目录教程

- `base.py`：基础设置查询和更新接口。
- `__init__.py`：包声明。

注意区分数据库中的动态系统设置与 `.env` 启动配置：前者可由 UI 改，后者通常重启生效。

`download_excel` 接收受控 FileRequest，从 `settings.EXCEL_PATH` 返回文件。核心安全点是规范化并验证路径仍在允许目录、拒绝 `..`/绝对路径、设置正确 Content-Disposition，并用 FileResponse/StreamingResponse 处理大文件。

## 与其他模块的调用关系

data_training、datasource 等模块生成/导出 Excel 后，可以返回逻辑文件标识，再由本接口下载。文件生命周期需要清理策略，避免临时导出永久占用磁盘。

## 错误语义

不存在返回 404，无权限返回 403，非法路径返回 400；不要用 500 混在一起。下载响应不进入普通 `code/data` JSON 包装。

