# frontend/public 静态文件模块

Vite 会把该目录原样复制到构建产物，不经过模块打包。主要内容是 TinyMCE 运行时资源、语言包、皮肤、主题和私有扩展资源。

`tinymce/` 是编辑器官方/裁剪资源，`tinymce-dataease-private/` 是项目定制资源。代码通过固定 URL 动态加载它们，因此目录结构和文件名属于运行时契约。

升级 TinyMCE 时要同步 npm 版本、Vue wrapper、plugins、langs、skins 和 CSP；不要手工修改 minified 文件来实现业务逻辑。静态目录中的文件会被公开访问，绝不能放 token、配置密码或用户上传数据。

