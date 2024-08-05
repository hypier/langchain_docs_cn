---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/ollama.ipynb
sidebar_label: Ollama
---

# OllamaEmbeddings

本笔记本介绍如何开始使用Ollama嵌入模型。

## 安装

# 安装包
%pip install langchain_ollama

## 设置

首先，按照[这些说明](https://github.com/jmorganca/ollama)设置并运行本地的Ollama实例：

* [下载](https://ollama.ai/download)并在可用的支持平台上安装Ollama（包括Windows子系统Linux）
* 通过`ollama pull <name-of-model>`获取可用的LLM模型
    * 通过[模型库](https://ollama.ai/library)查看可用模型列表
    * 例如，`ollama pull llama3`
* 这将下载模型的默认标签版本。通常，默认版本指向最新、参数最小的模型。

> 在Mac上，模型将下载到`~/.ollama/models`
> 
> 在Linux（或WSL）上，模型将存储在`/usr/share/ollama/.ollama/models`

* 指定感兴趣的模型的确切版本，如`ollama pull vicuna:13b-v1.5-16k-q4_0`（在此实例中查看`Vicuna`模型的[各种标签](https://ollama.ai/library/vicuna/tags)）
* 要查看所有已拉取的模型，请使用`ollama list`
* 要直接从命令行与模型聊天，请使用`ollama run <name-of-model>`
* 查看[Ollama文档](https://github.com/jmorganca/ollama)以获取更多命令。在终端中运行`ollama help`也可以查看可用命令。

## 使用方法


```python
from langchain_ollama import OllamaEmbeddings

embeddings = OllamaEmbeddings(model="llama3")
```


```python
embeddings.embed_query("我的查询")
```



```output
[1.1588108539581299,
 -3.3943021297454834,
 0.8108075261116028,
 0.48006290197372437,
 -1.8064439296722412,
 -0.5782400965690613,
 1.8570188283920288,
 2.2842330932617188,
 -2.836144208908081,
 -0.6422690153121948,
 ...]
```



```python
# 异步嵌入文档
await embeddings.aembed_documents(
    ["这是文档的内容", "这是另一个文档"]
)
```



```output
[[0.026717308908700943,
  -3.073253870010376,
  -0.983579158782959,
  -1.3976373672485352,
  0.3153868317604065,
  -0.9198529124259949,
  -0.5000395178794861,
  -2.8302183151245117,
  0.48412731289863586,
  -1.3201743364334106,
  ...]]
```

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)