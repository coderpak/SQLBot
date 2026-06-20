# ai_model/openai 目录教程

## 作用与文件

- `llm.py`：把 SQLBot 的模型配置映射为 LangChain OpenAI chat model，处理 API base、key、model、额外参数与流式输出。多数国内服务商通过 OpenAI-compatible 协议复用这里。
- `__init__.py`：包声明。

调试模型连接时，先核对管理界面保存的 protocol/supplier，再在本文件观察最终构造参数。

## 适配职责

实现把 `LLMConfig` 映射到 LangChain OpenAI/Azure/vLLM 客户端，包括 base_url、api_key、model、超时、重试、流式、temperature 与 additional_params。兼容供应商往往对空参数、extra_body 和 thinking 字段要求不同，构造时应只传它真正支持的参数。

## 输出契约

无论供应商原始事件格式如何，最终要让 `process_stream` 取得统一的 content、reasoning_content 和 token usage。供应商升级 SDK 后最容易破坏的就是流式 chunk 与 usage 字段，应有集成测试。

