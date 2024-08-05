---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_transformers/markdownify.ipynb
---

# Markdownify

> [markdownify](https://github.com/matthewwithanm/python-markdownify) 是一个 Python 包，可以将 HTML 文档转换为 Markdown 格式，并提供可自定义的选项来处理标签（链接、图片等）、标题样式和其他内容。

```python
%pip install --upgrade --quiet  markdownify
```

```python
from langchain_community.document_loaders import AsyncHtmlLoader

urls = ["https://lilianweng.github.io/posts/2023-06-23-agent/"]
loader = AsyncHtmlLoader(urls)
docs = loader.load()
```
```output
/Users/f.sokolov/Desktop/langchain/.venv/lib/python3.9/site-packages/urllib3/__init__.py:35: NotOpenSSLWarning: urllib3 v2 only supports OpenSSL 1.1.1+, currently the 'ssl' module is compiled with 'LibreSSL 2.8.3'. See: https://github.com/urllib3/urllib3/issues/3020
  warnings.warn(
Fetching pages: 100%|##########| 1/1 [00:00<00:00,  1.96it/s]
```

```python
docs
```

```output
```

```python
from langchain_community.document_transformers import MarkdownifyTransformer
```

```python
md = MarkdownifyTransformer()
converted_docs = md.transform_documents(docs)

print(converted_docs[0].page_content[:1000])
```
```output
LLM Powered Autonomous Agents | Lil'Log

[Lil'Log](https://lilianweng.github.io/ "Lil'Log (Alt + H)")

* [Posts](https://lilianweng.github.io/ "Posts")
* [Archive](https://lilianweng.github.io/archives "Archive")
* [Search](https://lilianweng.github.io/search/ "Search (Alt + /)")
* [Tags](https://lilianweng.github.io/tags/ "Tags")
* [FAQ](https://lilianweng.github.io/faq "FAQ")
* [emojisearch.app](https://www.emojisearch.app/ "emojisearch.app")

# LLM Powered Autonomous Agents

Date: June 23, 2023 | Estimated Reading Time: 31 min | Author: Lilian Weng

Table of Contents

* [Agent System Overview](#agent-system-overview)
* [Component One: Planning](#component-one-planning)
	+ [Task Decomposition](#task-decomposition)
	+ [Self-Reflection](#self-reflection)
* [Component Two: Memory](#component-two-memory)
	+ [Types of Memory](#types-of-memory)
	+ [Maximum Inner Product Search (MIPS)](#maximum-inner-product-search-mips)
* [Component Three: Tool Use](#component-three-tool-use)
* [Case Studi
```

```python
md = MarkdownifyTransformer(strip="a")
converted_docs = md.transform_documents(docs)

print(converted_docs[0].page_content[:1000])
```
```output
LLM Powered Autonomous Agents | Lil'Log

Lil'Log

* Posts
* Archive
* Search
* Tags
* FAQ
* emojisearch.app

# LLM Powered Autonomous Agents

Date: June 23, 2023 | Estimated Reading Time: 31 min | Author: Lilian Weng

Table of Contents

* Agent System Overview
* Component One: Planning
	+ Task Decomposition
	+ Self-Reflection
* Component Two: Memory
	+ Types of Memory
	+ Maximum Inner Product Search (MIPS)
* Component Three: Tool Use
* Case Studies
	+ Scientific Discovery Agent
	+ Generative Agents Simulation
	+ Proof-of-Concept Examples
* Challenges
* Citation
* References

Building agents with LLM (large language model) as its core controller is a cool concept. Several proof-of-concepts demos, such as AutoGPT, GPT-Engineer and BabyAGI, serve as inspiring examples. The potentiality of LLM extends beyond generating well-written copies, stories, essays and programs; it can be framed as a powerful general problem solver.

# Agent System Overview#

In a LLM-powered autonomous agent system
```

```python
md = MarkdownifyTransformer(strip=["h1", "a"])
converted_docs = md.transform_documents(docs)

print(converted_docs[0].page_content[:1000])
```
```output
LLM Powered Autonomous Agents | Lil'Log

Lil'Log

* Posts
* Archive
* Search
* Tags
* FAQ
* emojisearch.app

 LLM Powered Autonomous Agents

Date: June 23, 2023 | Estimated Reading Time: 31 min | Author: Lilian Weng

Table of Contents

* Agent System Overview
* Component One: Planning
	+ Task Decomposition
	+ Self-Reflection
* Component Two: Memory
	+ Types of Memory
	+ Maximum Inner Product Search (MIPS)
* Component Three: Tool Use
* Case Studies
	+ Scientific Discovery Agent
	+ Generative Agents Simulation
	+ Proof-of-Concept Examples
* Challenges
* Citation
* References

Building agents with LLM (large language model) as its core controller is a cool concept. Several proof-of-concepts demos, such as AutoGPT, GPT-Engineer and BabyAGI, serve as inspiring examples. The potentiality of LLM extends beyond generating well-written copies, stories, essays and programs; it can be framed as a powerful general problem solver.

Agent System Overview#
In a LLM-powered autonomous agent system, LL
```