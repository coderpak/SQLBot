# views/login 登录模块教程

`index.vue` 提供本地用户名密码登录；`xpack/` 包含 LDAP、OIDC、OAuth2、CAS、钉钉、飞书、企业微信和二维码等扩展认证 UI/回调处理。

成功登录后保存 token、加载用户信息/工作空间并由路由守卫跳转。第三方认证前端只负责发起流程和接收回调，code/token 校验、用户映射和签发 SQLBot token 必须在后端完成。回调参数不能直接信任，重定向地址要使用白名单。

