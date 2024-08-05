---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/llmsherpa.ipynb
---

# LLM Sherpa

本笔记本介绍如何使用 `LLM Sherpa` 加载多种类型的文件。`LLM Sherpa` 支持不同的文件格式，包括 DOCX、PPTX、HTML、TXT 和 XML。

`LLMSherpaFileLoader` 使用 LayoutPDFReader，这是 LLMSherpa 库的一部分。该工具旨在解析 PDF，同时保留其布局信息，这在使用大多数 PDF 转文本解析器时常常会丢失。

LayoutPDFReader 的一些关键特性：

* 它可以识别和提取章节和子章节及其层级。
* 它可以将行合并成段落。
* 它可以识别章节和段落之间的链接。
* 它可以提取表格以及表格所在的章节。
* 它可以识别和提取列表和嵌套列表。
* 它可以连接跨页的内容。
* 它可以删除重复的页眉和页脚。
* 它可以删除水印。

查看 [llmsherpa](https://llmsherpa.readthedocs.io/en/latest/) 文档。

`INFO: this library fail with some pdf files so use it with caution.`


```python
# Install package
# !pip install --upgrade --quiet llmsherpa
```

## LLMSherpaFileLoader

在底层，LLMSherpaFileLoader 定义了一些策略来加载文件内容：["sections", "chunks", "html", "text"]，设置 [nlm-ingestor](https://github.com/nlmatics/nlm-ingestor) 以获取 `llmsherpa_api_url` 或使用默认值。

### sections strategy: 将文件解析为各个部分


```python
from langchain_community.document_loaders.llmsherpa import LLMSherpaFileLoader

loader = LLMSherpaFileLoader(
    file_path="https://arxiv.org/pdf/2402.14207.pdf",
    new_indent_parser=True,
    apply_ocr=True,
    strategy="sections",
    llmsherpa_api_url="http://localhost:5010/api/parseDocument?renderFormat=all",
)
docs = loader.load()
```


```python
docs[1]
```



```output
Document(page_content='摘要\n我们研究如何应用大型语言模型从头开始撰写扎实且有组织的长篇文章，其广度和深度可与维基百科页面相媲美。\n这一尚未充分探索的问题在写作前阶段提出了新的挑战，包括如何研究主题和在写作前准备大纲。\n我们提出了STORM，这是一种通过\n参考文献合成主题大纲的写作系统。\n完整文章\n主题\n大纲\n2022年冬季奥运会\n开幕式\n通过提问进行研究\n检索和多角度提问。\nSTORM通过以下方式建模写作前阶段：\n(1) 在研究给定主题时发现多样化的观点，(2) 模拟对话，其中持有不同观点的作者向基于可信互联网来源的主题专家提问，(3) 策划收集的信息以创建大纲。\n为了评估，我们策划了FreshWiki，这是一个包含近期高质量维基百科文章的数据集，并制定大纲评估来评估写作前阶段。\n我们进一步收集了经验丰富的维基百科编辑的反馈。\n与由大纲驱动的检索增强基线生成的文章相比，更多的STORM文章被认为是有组织的（绝对增加25%）和覆盖广泛（增加10%）。\n专家反馈还帮助识别生成扎实长篇文章的新挑战，例如来源偏见转移和无关事实的过度关联。\n1. 能否提供有关开幕式的交通安排的任何信息？\nLLM\n2. 能否提供有关2022年冬季奥运会开幕式预算的任何信息？…\nLLM- Role1\nLLM- Role2\nLLM- Role1', metadata={'source': 'https://arxiv.org/pdf/2402.14207.pdf', 'section_number': 1, 'section_title': '摘要'})
```



```python
len(docs)
```



```output
79
```

### chunks strategy: 将文件解析为多个块


```python
from langchain_community.document_loaders.llmsherpa import LLMSherpaFileLoader

loader = LLMSherpaFileLoader(
    file_path="https://arxiv.org/pdf/2402.14207.pdf",
    new_indent_parser=True,
    apply_ocr=True,
    strategy="chunks",
    llmsherpa_api_url="http://localhost:5010/api/parseDocument?renderFormat=all",
)
docs = loader.load()
```


```python
docs[1]
```



```output
Document(page_content='Assisting in Writing Wikipedia-like Articles From Scratch with Large Language Models\nStanford University {shaoyj, yuchengj, tkanell, peterxu, okhattab}@stanford.edu lam@cs.stanford.edu', metadata={'source': 'https://arxiv.org/pdf/2402.14207.pdf', 'chunk_number': 1, 'chunk_type': 'para'})
```



```python
len(docs)
```



```output
306
```

### html 策略：将文件作为一个 HTML 文档返回


```python
from langchain_community.document_loaders.llmsherpa import LLMSherpaFileLoader

loader = LLMSherpaFileLoader(
    file_path="https://arxiv.org/pdf/2402.14207.pdf",
    new_indent_parser=True,
    apply_ocr=True,
    strategy="html",
    llmsherpa_api_url="http://localhost:5010/api/parseDocument?renderFormat=all",
)
docs = loader.load()
```


```python
docs[0].page_content[:400]
```



```output
'<html><h1>Assisting in Writing Wikipedia-like Articles From Scratch with Large Language Models</h1><table><th><td colSpan=1>Yijia Shao</td><td colSpan=1>Yucheng Jiang</td><td colSpan=1>Theodore A. Kanell</td><td colSpan=1>Peter Xu</td></th><tr><td colSpan=1></td><td colSpan=1>Omar Khattab</td><td colSpan=1>Monica S. Lam</td><td colSpan=1></td></tr></table><p>Stanford University {shaoyj, yuchengj, '
```



```python
len(docs)
```



```output
1
```

### 文本策略：将文件作为一个文本文档返回


```python
from langchain_community.document_loaders.llmsherpa import LLMSherpaFileLoader

loader = LLMSherpaFileLoader(
    file_path="https://arxiv.org/pdf/2402.14207.pdf",
    new_indent_parser=True,
    apply_ocr=True,
    strategy="text",
    llmsherpa_api_url="http://localhost:5010/api/parseDocument?renderFormat=all",
)
docs = loader.load()
```


```python
docs[0].page_content[:400]
```



```output
'Assisting in Writing Wikipedia-like Articles From Scratch with Large Language Models\n | Yijia Shao | Yucheng Jiang | Theodore A. Kanell | Peter Xu\n | --- | --- | --- | ---\n |  | Omar Khattab | Monica S. Lam | \n\nStanford University {shaoyj, yuchengj, tkanell, peterxu, okhattab}@stanford.edu lam@cs.stanford.edu\nAbstract\nWe study how to apply large language models to write grounded and organized long'
```



```python
len(docs)
```



```output
1
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)