# g2-ssr/charts 图表渲染模块

## 文件职责

- `bar.js`：横向条形图 SSR 配置。
- `column.js`：纵向柱状图 SSR 配置。
- `line.js`：折线图 SSR 配置。
- `pie.js`：饼图 SSR 配置。
- `utils.js`：公共画布、主题、字段/数值转换、图片输出辅助。

## 输入契约

每个实现接收规范化后的 axis 与 data。x/y/series 的 value 必须对应数据行字段；空数据、空轴和非数值指标要明确回退。图表类型来自后端模型输出，但 app.js 只应分派白名单类型。

## 与浏览器实现的关系

这里和 `frontend/src/views/chat/component/charts` 是两套渲染实现，共享概念但运行环境不同。修改颜色、轴逻辑、multi-quota 或新图表类型时要同步两端，并用同一份 fixture 对比图片和浏览器图表。

