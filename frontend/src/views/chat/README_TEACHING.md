# views/chat 模块教程

## 核心作用

把后端一轮长时间运行的问数任务呈现为可交互的渐进式回答：先显示用户问题，再展示 SQL 推理/SQL、查询数据、图表推理和最终图表，同时允许停止、重新生成、分析、预测、导出和查看执行日志。

## 组件层次

- `index.vue`：页面总协调、会话与 records、输入发送、滚动和回答选择。
- `ChatCreator/ChatList*`：新会话与历史会话。
- `answer/`：Chart/Analysis/Predict 三类流式回答，共享 BaseAnswer。
- `chat-block/`：用户消息、图表卡片、工具栏。
- `component/`：Markdown、SQL、图表抽象和工厂。
- `execution-component/`：把后端 ChatLog 的选表、术语、样例、AI、查询步骤可视化。
- `Recent/Recommend/QuickQuestion`：问题快捷入口。

## 状态来源

会话与 record 主要在页面响应式状态中；身份/嵌入模式来自 assistant store；是否显示 SQL 等来自 chatConfig；真实历史通过 chatApi 加载。临时 record 在拿到后端 id 前也要渲染，因此逻辑必须能处理 undefined id。

完整事件协议和图表链路见 `CHAT_UI_TUTORIAL.md`。

