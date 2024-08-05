---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/recursive_text_splitter.ipynb
keywords: [recursivecharactertextsplitter]
---

# 如何通过字符递归拆分文本

这个文本拆分器是推荐用于通用文本的。它通过字符列表进行参数化。它尝试按顺序在这些字符上进行拆分，直到文本块足够小。默认列表为 `["\n\n", "\n", " ", ""]`。这会尽量保持所有段落（然后是句子，接着是单词）尽可能地在一起，因为这些通常看起来是语义上最强相关的文本片段。

1. 文本是如何拆分的：通过字符列表。
2. 块大小是如何测量的：通过字符数。

下面我们展示示例用法。

要直接获取字符串内容，请使用 `.split_text`。

要创建 LangChain [Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html) 对象（例如，用于下游任务），请使用 `.create_documents`。


```python
%pip install -qU langchain-text-splitters
```


```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

# 加载示例文档
with open("state_of_the_union.txt") as f:
    state_of_the_union = f.read()

text_splitter = RecursiveCharacterTextSplitter(
    # 设置一个非常小的块大小，仅用于展示。
    chunk_size=100,
    chunk_overlap=20,
    length_function=len,
    is_separator_regex=False,
)
texts = text_splitter.create_documents([state_of_the_union])
print(texts[0])
print(texts[1])
```
```output
page_content='Madam Speaker, Madam Vice President, our First Lady and Second Gentleman. Members of Congress and'
page_content='of Congress and the Cabinet. Justices of the Supreme Court. My fellow Americans.'
```

```python
text_splitter.split_text(state_of_the_union)[:2]
```



```output
['Madam Speaker, Madam Vice President, our First Lady and Second Gentleman. Members of Congress and',
 'of Congress and the Cabinet. Justices of the Supreme Court. My fellow Americans.']
```


让我们来看看上面为 `RecursiveCharacterTextSplitter` 设置的参数：
- `chunk_size`：块的最大大小，大小由 `length_function` 决定。
- `chunk_overlap`：块之间的目标重叠。重叠的块有助于减轻在块之间划分上下文时信息的丢失。
- `length_function`：确定块大小的函数。
- `is_separator_regex`：分隔符列表（默认为 `["\n\n", "\n", " ", ""]`）是否应被解释为正则表达式。

## 从没有词边界的语言中拆分文本

某些书写系统没有[词边界](https://en.wikipedia.org/wiki/Category:Writing_systems_without_word_boundaries)，例如中文、日文和泰文。使用默认的分隔符列表 `["\n\n", "\n", " ", ""]` 拆分文本可能会导致单词在块之间被拆分。为了保持单词的完整性，可以覆盖分隔符列表以包含额外的标点符号：

* 添加 ASCII 句号 "`.`"，[Unicode 全宽](https://en.wikipedia.org/wiki/Halfwidth_and_Fullwidth_Forms_(Unicode_block)) 句号 "`．`"（用于中文），以及 [表意全句号](https://en.wikipedia.org/wiki/CJK_Symbols_and_Punctuation) "`。`"（用于日文和中文）
* 添加在泰文、缅甸文、柬文和日文中使用的 [零宽空格](https://en.wikipedia.org/wiki/Zero-width_space)。
* 添加 ASCII 逗号 "`,`"，Unicode 全宽逗号 "`，`" 和 Unicode 表意逗号 "`、`"


```python
text_splitter = RecursiveCharacterTextSplitter(
    separators=[
        "\n\n",
        "\n",
        " ",
        ".",
        ",",
        "\u200b",  # Zero-width space
        "\uff0c",  # Fullwidth comma
        "\u3001",  # Ideographic comma
        "\uff0e",  # Fullwidth full stop
        "\u3002",  # Ideographic full stop
        "",
    ],
    # Existing args
)
```