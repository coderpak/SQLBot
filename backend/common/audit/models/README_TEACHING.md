# common/audit/models 目录教程

- `log_model.py`：系统操作日志表、OperationType、OperationModules 等枚举。
- `__init__.py`：包声明。

`SystemLog` 保存用户/空间、模块、操作、资源、请求、状态、耗时、客户端和错误等字段；OperationType/Modules/Status 枚举保证不同 API 使用统一分类。User-Agent、detail 和 error 可能很长，字段长度与迁移要匹配，且敏感值在写入前脱敏。

## 查询用途

前端审计页面可按时间、用户、模块、操作和状态筛选。为常用过滤字段建索引；detail/error 等大文本不要参与普通列表 select，详情时再加载。

## 保留与合规

审计表会持续增长，需要归档/保留周期。日志自身也是敏感数据，查看和导出应有管理员权限，并避免记录业务查询结果全集。

