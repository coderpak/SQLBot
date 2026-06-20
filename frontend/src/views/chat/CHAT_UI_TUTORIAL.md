# AI 问数前端逐文件教程

## 先抓住数据模型

一次问答在前端表现为 `ChatInfo.records` 中的一个 `ChatRecord`。记录里最重要的是：`question`、`sql_answer`、`sql`、`data`、`chart_answer`、`chart`、`error`、`finish`。其中 answer 是模型推理/原始回答，`sql` 与 `chart` 才是校验后用于 UI 的结果。

## 核心文件

| 文件 | 作用 |
|---|---|
| `index.vue` | 聊天总页；输入、会话切换、临时记录、滚动和回答组件编排。`sendMessage` 是用户动作入口。 |
| `answer/ChartAnswer.vue` | 发起问数流式请求并解析所有 SSE 事件，是前端链路的中心。 |
| `chat-block/ChartBlock.vue` | 把后端的 chart JSON 和 data 组装为可视化，还负责切图、查看 SQL、导出、全屏、加入仪表板。 |
| `component/DisplayChartBlock.vue` | 将 `axis.x/y/series/multi-quota` 规范成数组，传给通用图表组件。 |
| `component/ChartComponent.vue` | 根据 type 创建图表类，调用 `init`、`render`、`destroy`。 |
| `component/index.ts` | `CHART_TYPE_MAP` 工厂：table/bar/column/line/pie → 对应类。 |
| `component/BaseChart.ts` | 所有图表的抽象接口和 axis/data 类型。 |
| `component/BaseG2Chart.ts` | G2 图表公共生命周期。 |
| `component/charts/*.ts` | 各图表的 G2/S2 具体编码。 |
| `../../api/chat.ts` | Chat/ChatRecord 类型、普通聊天 API、`questionApi.add`。 |
| `../../utils/request.ts` | Axios 拦截器与单独的 `fetchStream` 实现。 |

## SSE 解析要点

HTTP chunk 不保证恰好等于一条事件，因此 `ChartAnswer.vue` 使用 `tempResult` 缓存残片，再按 `data:...\n\n` 拆包。解析时使用 `json-bigint`，避免 Snowflake ID 被 JavaScript Number 截断。收到 `sql-result`/`chart-result` 时追加文本；`sql-data` 只是完成信号，实际 rows 再由 `chatApi.get_chart_data` 获取。

## 图表 JSON 契约

典型结构：

```json
{
  "type": "column",
  "title": "各地区销售额",
  "columns": [{"name": "地区", "value": "region"}],
  "axis": {
    "x": {"name": "地区", "value": "region"},
    "y": {"name": "销售额", "value": "sales"},
    "series": null
  }
}
```

`value` 必须与数据行的字段 key 一致。新增图表类型时要同步更新 `ChartTypes`、`CHART_TYPE_MAP`、具体图表类、后端 chart prompt 的允许类型和必要轴规则。

## 建议断点

1. `index.vue::sendMessage`：看临时记录如何进入 records。
2. `ChartAnswer.vue` 的 `switch(data.type)`：看事件如何逐步改变同一条记录。
3. `getChatData`：确认 `sql-data` 后数据接口响应。
4. `ChartComponent.vue::renderChart`：观察最终 axis/data。

## 组件调用树

```text
chat/index.vue
├─ ChatListContainer / ChatCreator
├─ ChatRow
│  ├─ UserChat
│  └─ ChartAnswer / AnalysisAnswer / PredictAnswer
│     └─ BaseAnswer
│        └─ ChartBlock
│           └─ DisplayChartBlock
│              └─ ChartComponent
│                 └─ Table / Bar / Column / Line / Pie
└─ ExecutionDetails
   └─ LogChooseTable / LogTerm / LogSQLSample / LogWithAi / LogDataQuery ...
```

index 管“有哪些会话和记录”，Answer 管“这一条记录怎样流式完成”，ChartBlock 管“完成后怎样使用结果”，ExecutionDetails 管“为什么得到这个结果”。

## 用户点击发送后的真实步骤

1. `index.vue::sendMessage` 忽略输入法 composition 和空白问题。
2. 设置 loading/isTyping，启动自动滚动。
3. `assistantPrepareSend` 处理嵌入助手前置身份。
4. 创建尚无 id 的 ChatRecord，写入 question/chat_id/regenerate_record_id。
5. 把临时记录加入当前 ChatInfo.records，并清空输入框。
6. 让对应 ChartAnswer 通过 expose 的 `sendMessage()` 发起请求。
7. 后端返回 id 后，更新同一条响应式 record，而不是创建另一条。

临时记录使用户点击后立即看到消息，但所有使用 record.id 的按钮必须处理 undefined。

## `fetchStream` 为什么不使用 Axios

Axios 封装适合普通 JSON，但浏览器原生 `fetch` 能直接读取 `response.body.getReader()`。`questionApi.add` 只是薄封装，真正 URL、Header 和 AbortSignal 在 request.ts 中设置。注意它不会经过 Axios interceptor，所以 token、assistant certificate、host-origin 等逻辑在 fetchStream 中单独实现。

## SSE 拆包算法

网络层可能这样切块：

```text
chunk1: data:{"type":"sql-res
chunk2: ult","reasoning_content":"..."}\n\ndata:{"type":"sql"
chunk3: ,"content":"SELECT ..."}\n\n
```

因此不能对每个 reader.read() 直接 JSON.parse。ChartAnswer 把文本加到 `tempResult`，用完整事件正则取出一个或多个 `data:...\n\n`，剩余半包继续等待。每条事件用 JSONBig 解析。

更稳健的扩展实现可按空行逐帧，并处理多行 `data:`、CRLF 和 decoder flush；修改时应加入“一个 chunk 多事件、一个事件多 chunk、中文跨字节边界”测试。

## 每种事件怎样改变 record

| type | 修改字段/状态 |
|---|---|
| `id` | `record.id`，后续数据、日志、导出都依赖它 |
| `regenerate_record_id` | 建立重生成关系 |
| `question` | 使用后端最终保存的问题 |
| `brief` | 更新当前 Chat 与左侧 ChatList 标题 |
| `sql-result` | 追加到 `sql_answer`，展示推理过程 |
| `sql` | 保存格式化后的安全 SQL |
| `sql-data` | 调用 `getChatData(record.id)` |
| `chart-result` | 追加到 `chart_answer` |
| `chart` | 保存 JSON 字符串，触发图表计算属性 |
| `datasource` | 自动选择时回填 Chat.datasource |
| `error` | 写 error、停止 loading、触发 error emit |
| `finish` | 触发父组件收尾、推荐问题等逻辑 |

每次处理后 `await nextTick()`，让长流中的 UI 渐进更新。

## 为什么 `data` 单独请求

SQL 结果可能有大量行，放进 SSE 会让事件解析、重连和内存更糟。后端保存数据并发 `sql-data` 信号，前端用 `/chat/record/{id}/data` 获取标准 `{fields,data}`。历史会话加载时也能按需取数据。

`getChatData` 遍历当前 records 找相同 id 并赋值。若用户快速切换会话，要防旧请求回来修改不再活动的对象；可以用 record id 和 chat id 双重确认或取消请求。

## 图表配置怎样变成 G2/S2

1. `ChartBlock.chartObject` JSON.parse record.chart。
2. `dataObject` 取得 record.data，预测场景可拼接 predict_data。
3. ChartBlock 选择当前 chart type，并提供切表格、全屏、标签、SQL、导出和加入看板。
4. DisplayChartBlock 把 axis.x/y/series 统一成数组，为 multi-quota 标记 y 指标。
5. ChartComponent 合并 columns 与 axis，调用 `getChartInstance(type,id)`。
6. 工厂只从 CHART_TYPE_MAP 实例化受支持类。
7. `init(axis,data)` 保存输入，`render()` 构建 G2 或 S2；组件卸载时 `destroy()`。

## 五种图表类

- `Table`：适合明细和多维数据，使用 S2/表格能力并处理排序。
- `Column`：类别 x + 数值 y，纵向比较。
- `Bar`：类别较长时横向比较。
- `Line`：时间/有序维度趋势。
- `Pie`：少量类别占比，过多类别可读性差。

后端建议类型不等于强制类型；ChartBlock 对 column/bar/line 允许用户切换，table 是通用回退。

## SQL、导出与看板

ChartBlock 的 SQL drawer 使用 SQLComponent 高亮最终 SQL；Excel 导出调用后端，避免浏览器仅导出当前截断数据；图片导出用 html2canvas 捕获当前图表；加入看板把 type、title、columns、x/y/series、data、sql、datasource 转换为 dashboard view。

这些动作都依赖 record 已完成且有 id。权限变化后，后端导出/看板查询仍应重新校验，不能只信前端按钮是否显示。

## 停止生成

Answer 创建 AbortController，`stop()` 设置 flag 并 abort；组件卸载也调用 stop，避免页面切换后继续读取流。abort 只保证浏览器不再等待，后端线程是否立刻停止取决于模型 SDK 和任务实现，因此服务端仍需超时与资源释放。

## 大整数处理

Chat/Record ID 是 Snowflake BigInteger，普通 `JSON.parse` 会丢精度。Axios response transform 和 SSE 都使用 JSONBig；不要在组件里 `Number(record.id)`，DOM id 可以直接拼字符串。

## 常见故障排查

| 现象 | 检查 |
|---|---|
| 点击无请求 | composition、空输入、loading、Answer ref |
| 401 | fetchStream 是否带正确 user/assistant Header |
| 一直转圈 | reader 是否 done、事件正则是否留下完整帧、error 是否处理 |
| SQL 推理有但无 SQL | 后端解析/安全校验可能 error |
| 收到 sql-data 但空图 | data API 是否成功、records 中 id 是否匹配 |
| chart 有但不渲染 | JSON 解析、type 白名单、axis.value/data key |
| ID 末位变化 | 某处使用了原生 JSON.parse/Number |
| 切会话后串数据 | 未取消旧流或异步数据回写缺少 chat 校验 |

## 新增 SSE 事件的完整步骤

后端 run_task yield 新 type → API 保持流式 → ChatAnswer switch 处理并更新 record → ChatRecord 类型增加字段 → 历史接口/转换函数支持恢复 → UI 组件显示 → error/stop/重连场景测试。

## 新增图表类型的完整步骤

1. 后端 chart prompt 增加类型和轴规则。
2. 后端 chart schema 校验允许该类型。
3. `BaseChart.ts::ChartTypes` 增加字面量。
4. 新建 `component/charts/X.ts` 实现 render/destroy。
5. `component/index.ts::CHART_TYPE_MAP` 注册。
6. ChartBlock 添加图标、切换和导出策略。
7. g2-ssr 添加同类型图片渲染。
8. 使用相同 chart/data fixture 做浏览器与 SSR 测试。

