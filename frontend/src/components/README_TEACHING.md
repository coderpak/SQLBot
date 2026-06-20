# frontend/src/components 通用组件教程

该目录存放跨页面复用的 UI 构件：layout（整体菜单/个人/工作空间）、drawer-main/filter（通用抽屉）、filter-text、icon-custom、Language-selector、rich-text 和 about。

通用组件通过 props/emit/slot 接收业务数据，不应直接硬编码 datasource/chat API。layout 是例外，它负责应用壳和用户/工作空间全局动作。富文本基于 TinyMCE，渲染外部内容时仍需关注 XSS 和上传资源权限。

