# datasource/embedding 目录教程

## 文件

- `table_embedding.py`：把问题向量与表 embedding 比较，返回最相关的若干表。
- `ds_embedding.py`：未指定数据源时，从候选数据源中召回相关数据源。
- `utils.py`：向量相似度、序列化或公共计算辅助。
- `__init__.py`：包声明。

配置项 `TABLE_EMBEDDING_ENABLED`、`TABLE_EMBEDDING_COUNT`、`DS_EMBEDDING_COUNT` 控制召回。关闭后通常会把更多元数据交给模型，可能更慢且噪声更大。

## 表召回

每张表的 embedding 通常由表名、注释和字段信息生成。`calc_table_embedding` 计算问题向量与候选表向量的相似度，选 Top-N；关系模块随后可能补入连接所需的邻表。因此 Top-N 不是最终表数。

## 数据源召回

没有预选数据源或动态助手场景下，ds_embedding 在授权候选集内按描述/元数据相似度选数据源。召回结果只是候选，还要验证归属与连接。

## 性能与一致性

Embedding 模型首次加载昂贵，应复用实例；表注释/字段变化后要更新向量；更换 embedding 模型会让旧新向量不在同一空间，通常需要全量重建。

