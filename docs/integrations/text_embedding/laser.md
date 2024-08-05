---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/laser.ipynb
---

# LASER 语言无关句子表示嵌入由 Meta AI 提供

>[LASER](https://github.com/facebookresearch/LASER/) 是由 Meta AI 研究团队开发的一个 Python 库，用于为截至 2024 年 2 月 25 日的 147 种语言创建多语言句子嵌入
>- 支持的语言列表请参见 https://github.com/facebookresearch/flores/blob/main/flores200/README.md#languages-in-flores-200

## 依赖关系

要将 LaserEmbed 与 LangChain 一起使用，请安装 `laser_encoders` Python 包。

```python
%pip install laser_encoders
```

## 导入


```python
from langchain_community.embeddings.laser import LaserEmbeddings
```

## 实例化激光

### 参数
- `lang: Optional[str]`
    >如果为空，将默认使用多语言 LASER 编码器模型（称为 "laser2"）。
    您可以在 [这里](https://github.com/facebookresearch/flores/blob/main/flores200/README.md#languages-in-flores-200) 和 [这里](https://github.com/facebookresearch/LASER/blob/main/laser_encoders/language_list.py) 找到支持的语言和 lang_codes 的列表。

```python
# Ex Instantiationz
embeddings = LaserEmbeddings(lang="eng_Latn")
```

## 使用方法

### 生成文档嵌入


```python
document_embeddings = embeddings.embed_documents(
    ["This is a sentence", "This is some other sentence"]
)
```

### 生成查询嵌入


```python
query_embeddings = embeddings.embed_query("This is a query")
```

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)