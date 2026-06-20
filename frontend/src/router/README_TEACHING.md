# frontend/src/router 模块教程

## 文件

- `index.ts`：hash 路由表与 Router 实例，包含登录、聊天、数据源、工作台、仪表板、嵌入和系统页面。
- `dynamic.ts`：根据用户权限/角色生成或过滤动态菜单路由。
- `watch.ts`：导航守卫、token/用户初始化、工作空间、标题和异常跳转。

## 核心作用

路由不仅把 URL 映射到组件，也是登录态和权限的第一层 UI 门。真正的安全仍由后端实现，前端隐藏菜单不能替代 API 授权。

## Hash History

项目使用 `createWebHashHistory`，URL 中 `#` 后的路径由前端处理，部署时不要求 Web 服务器为所有前端路径做 history fallback，适合嵌入和一体化镜像。

## 新页面步骤

创建 view → 注册静态或动态 route → 增加菜单/权限 key → 补 i18n → 确认嵌入模式是否可见 → 后端 API 仍设置权限。

