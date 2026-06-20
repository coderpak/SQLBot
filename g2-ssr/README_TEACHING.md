# g2-ssr 目录教程

这是 Node.js 图表服务端渲染模块，主要服务 MCP 或外部集成的“返回图表图片”场景；浏览器中的交互图表仍由 frontend 渲染。

## 文件

- `app.js`：3000 端口 HTTP 服务；解析 GET/POST，请求中的 type/axis/data 经 `getOptions` 规范化，再由 `GenerateCharts` 分派。
- `charts/bar.js`、`column.js`、`line.js`、`pie.js`：各图表的 G2 服务端配置与图片生成。
- `charts/utils.js`：坐标轴、主题、数据与渲染公共逻辑。
- `package.json`：G2、canvas/图片和 PM2 等依赖。
- `ecosystem.config.js`：PM2 进程配置。
- `Arial_Unicode.ttf`：容器内补充字体，避免中文/Unicode 变方框。
- `.gitignore`：Node 产物忽略规则。

## 请求链路

MCP 问数得到 chart JSON 与查询数据 → 后端调用 `MCP_IMAGE_HOST` 指向的 SSR 服务 → app.js 选择图表模块 → 生成图片写入/返回 → 后端保存到 `MCP_IMAGE_PATH` → 对外返回 `SERVER_IMAGE_HOST` URL。

## 与前端图表保持一致

支持类型和 axis 语义要与 frontend `CHART_TYPE_MAP`、后端 chart prompt 一致。新增图表类型需三端同步。SSR 无浏览器布局环境，字体、canvas 原生依赖和容器系统库是常见故障点。

## 安全与稳定

限制请求体和数据行数，校验 type 白名单，不允许请求指定任意输出路径。渲染失败应记录 chart type/字段而非完整敏感数据。Dockerfile 单独构建依赖，运行时由 PM2 托管。

