# frontend/src/entity 实体教程

- `CommonEntity.ts`：跨页面通用实体/分页或选项结构。
- `supplier.ts`：AI 模型供应商、协议或展示映射。

仅把真正跨多个 API/页面共享的稳定类型放这里；ChatRecord 等强业务实体留在对应 api 模块，避免形成无边界的全局 types 文件。

