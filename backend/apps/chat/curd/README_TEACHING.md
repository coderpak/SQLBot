# chat/curd 目录教程

> `curd` 是项目历史拼写，语义等同 CRUD。

## 文件

- `chat.py`：Chat、ChatRecord、ChatLog 的查询和更新；保存问题、SQL 原始回答、SQL、执行数据、图表回答、图表配置、分析/预测；读取历史；格式化 JSON；记录每步 token/耗时。
- `__init__.py`：包声明。

## 阅读方法

按 `save_question` → `save_sql_answer` → `save_sql` → `save_sql_exec_data` → `save_chart_answer` → `save_chart` → `finish_record` 阅读，它们正好对应一轮问数的持久化状态迁移。`format_json_list_data` 负责大数保护与字段名规范化，修改数据返回协议时必须一起验证前端。

## 读写对象

`Chat` 保存会话标题、数据源和推荐问题；`ChatRecord` 保存每轮 question/sql/data/chart/analysis/predict/error；`ChatLog` 保存操作步骤；查询函数常把数据源名称、类型和记录列表组装成 ChatInfo。

## 为什么分多个 save 函数

LLM 与数据库是长任务，不能等全部成功才一次 commit。每阶段持久化能在失败后保留现场、支持页面刷新、执行详情和重新生成。代价是它不是单个原子事务，状态字段必须能表达“部分完成”。

## 数据读取

图表数据可以存储/恢复为 JSON，再由 `get_chat_chart_data` 返回；`format_json_data` 统一 fields/data；超长整数转 string，限定字段名正规化，避免 SQL `table.column` 与 chart `column` 不匹配。

## 修改注意

新增 ChatRecord 字段要同步 model、migration、查询 select 列、toChatRecord 和历史接口。只改 ORM model 而忘记手写 select/response 转换，是该文件常见漏点。

