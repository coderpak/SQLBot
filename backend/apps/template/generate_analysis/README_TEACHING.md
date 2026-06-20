# template/generate_analysis 目录教程

- `generator.py`：读取“基于查询结果生成自然语言分析”的系统与用户模板。
- `__init__.py`：包声明。

分析阶段输入字段定义和已查询数据，输出面向用户的 Markdown/自然语言洞察，不再执行新的业务 SQL。模板应约束模型忠于数据、标明空值/样本限制，禁止凭空补数字。

## 输入

- 原始问题与查询字段。
- 已执行 SQL 返回的数据，通常已做大小和格式控制。
- 召回的术语与分析类自定义提示词。

## 输出与消费方

输出被 `LLMService.generate_analysis` 流式发送并保存到分析子记录，前端 `AnalysisAnswer.vue` 以 Markdown 展示。它不是 chart JSON，也不应被当作可执行 SQL。

## 调试

若结论数字不对，先比较传给 prompt 的 fields/data，而不是只看最终文字；若数据被截断，要让模型明确“基于前 N 行”。

