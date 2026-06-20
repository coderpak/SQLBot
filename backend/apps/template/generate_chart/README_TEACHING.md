# template/generate_chart 目录教程

- `generator.py`：`get_chart_template` 返回图表 JSON 生成模板，同时为术语和训练模块提供基础格式模板。
- `__init__.py`：包声明。

修改图表 JSON schema 时，必须同步 `LLMService.check_save_chart` 和前端 `DisplayChartBlock.vue`。

图表阶段输入 question、已确认 SQL、建议 chart_type 和实际使用表 Schema，输出 type/title/columns/axis。axis.value 最终会小写并与查询结果 key 匹配；不支持的类型应回退 table，而不是让前端动态执行模型给出的组件名。

## 支持类型

开源前端工厂支持 table、bar、column、line、pie。y 可以是单对象或数组，multi-quota 标记多指标；series 是分组维度。

## 校验

`check_save_chart` 从混合文本提取 JSON，检查 type/error，统一 columns/x/y/series/multi-quota 的 value 小写并保存。进一步扩展时建议增加字段存在性和类型白名单校验。

## 常见空图原因

模型 axis.value 使用了中文显示名而非 SQL alias；查询字段被规范化为小写；数据为空；y 指向文本列；生成了前端不支持的 type。

