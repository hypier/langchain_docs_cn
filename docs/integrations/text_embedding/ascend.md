---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/ascend.ipynb
---
```python
from langchain_community.embeddings import AscendEmbeddings

model = AscendEmbeddings(
    model_path="/root/.cache/modelscope/hub/yangjhchs/acge_text_embedding",
    device_id=0,
    query_instruction="为搜索相关段落表示此句子: ",
)
emb = model.embed_query("hellow")
print(emb)
```
```output
[-0.04053403 -0.05560051 -0.04385472 ...  0.09371872  0.02846981
 -0.00576814]
```

```python
doc_embs = model.embed_documents(
    ["这是文档的内容", "这是另一个文档"]
)
print(doc_embs)
```
```output
我们强烈建议传入 `attention_mask`，因为您的 input_ids 可能是填充的。请参见 https://huggingface.co/docs/transformers/troubleshooting#incorrect-output-when-padding-tokens-arent-masked.
``````output
[[-0.00348254  0.03098977 -0.00203087 ...  0.08492374  0.03970494
  -0.03372753]
 [-0.02198593 -0.01601127  0.00215684 ...  0.06065163  0.00126425
  -0.03634358]]
```

```python
model.aembed_query("hellow")
```



```output
<coroutine object Embeddings.aembed_query at 0x7f9fac699cb0>
```



```python
await model.aembed_query("hellow")
```



```output
array([-0.04053403, -0.05560051, -0.04385472, ...,  0.09371872,
        0.02846981, -0.00576814], dtype=float32)
```



```python
model.aembed_documents(
    ["这是文档的内容", "这是另一个文档"]
)
```



```output
<coroutine object Embeddings.aembed_documents at 0x7fa093ff1a80>
```



```python
await model.aembed_documents(
    ["这是文档的内容", "这是另一个文档"]
)
```



```output
array([[-0.00348254,  0.03098977, -0.00203087, ...,  0.08492374,
         0.03970494, -0.03372753],
       [-0.02198593, -0.01601127,  0.00215684, ...,  0.06065163,
         0.00126425, -0.03634358]], dtype=float32)
```

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)