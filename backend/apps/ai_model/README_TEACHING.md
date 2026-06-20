# ai_model 目录教程

## 作用

把数据库中的模型配置转换为 LangChain 可调用对象，并提供文本 embedding。

## 文件

- `model_factory.py`：模型配置读取与工厂，`get_default_config` 选择默认/指定模型，`LLMFactory` 创建适配器。
- `llm.py`：LLM 抽象、配置和通用调用封装。
- `embedding.py`：本地/远程 embedding 模型加载与向量生成。
- `openai/`：OpenAI 原生及 OpenAI-compatible 实现。
- `__init__.py`：包声明。

阅读连接：`LLMService.create` → `get_default_config` → `LLMFactory.create_llm`。

## 核心输入与输出

输入是系统库中的 `AiModelDetail`：supplier、model_type、protocol、base_model、api_domain、api_key 和 config。`get_default_config` 将其解密并转换成 `LLMConfig`；工厂输出一个 `BaseLLM` 包装器，其 `.llm` 是 LangChain `BaseChatModel`。chat 模块只调用统一的 `stream(messages)`，不关心供应商 SDK。

## 模型选择流程

1. 问数服务先取得当前工作空间可用模型列表。
2. 助手若启用 custom model 且该模型确实映射到工作空间，则使用指定模型。
3. 否则读取默认模型。
4. 根据 model_type/protocol 创建 OpenAI、Azure、vLLM 或兼容实现。
5. 额外参数被传给供应商 SDK；部分模型的 thinking 开关会按调用场景调整。

## Embedding 与 Chat Model 的区别

Chat Model 生成 SQL、图表和分析文本；Embedding Model 把问题、术语、SQL 示例、表说明变成向量。二者可以来自不同供应商，失败表现也不同。`embedding.py` 的模型加载通常在首次使用或启动补向量时最耗时。

## 扩展和调试

- OpenAI-compatible 服务优先复用现有实现，只增加配置，不急于写新类。
- 新协议需实现 `BaseLLM._init_llm` 并通过 `LLMFactory.register_llm` 注册。
- 调试顺序：管理界面 check_llm → `get_default_config` 最终配置 → SDK 请求 → `process_stream` chunk。
- 严禁打印完整 API Key；可记录模型 ID、供应商、base URL 脱敏值和响应状态。

