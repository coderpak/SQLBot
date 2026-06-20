# template/generate_guess_question 目录教程

- `generator.py`：生成推荐追问的 prompt 入口。
- `__init__.py`：包声明。

结果由 `save_recommend_question_answer` 从模型文本中提取 JSON 数组并保存。

输入当前问答和历史问题，输出有限数量的后续问题 JSON。问题应可由同一数据源回答、避免重复和越权字段。解析失败时回退空数组，不能阻断主问数流程。

## 触发与保存

主问数完成后或客户端显式请求时触发；LLMService 读取旧问题和当前上下文，模型返回列表，`save_recommend_question_answer` 保存原始回答和解析后的问题数组，必要时同步到 Chat。

## 前端使用

RecommendQuestion/RecommendQuestionQuick 展示问题，点击后回到正常 sendMessage 流程。推荐问题本身没有特殊权限，真正执行时仍走完整授权和 SQL 安全链。

## 质量标准

与当前主题相关、答案可由现有表获得、措辞清晰、彼此不重复、不会暗示不存在字段。

