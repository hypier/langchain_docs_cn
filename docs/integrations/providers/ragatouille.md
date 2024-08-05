---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/providers/ragatouille.ipynb
---

# RAGatouille

[RAGatouille](https://github.com/bclavie/RAGatouille) 使得使用 ColBERT 变得简单易行！ [ColBERT](https://github.com/stanford-futuredata/ColBERT) 是一种快速而准确的检索模型，能够在数十毫秒内对大文本集合进行可扩展的基于 BERT 的搜索。

我们可以通过多种方式使用 RAGatouille。

## 设置

集成位于 `ragatouille` 包中。

```bash
pip install -U ragatouille
```


```python
from ragatouille import RAGPretrainedModel

RAG = RAGPretrainedModel.from_pretrained("colbert-ir/colbertv2.0")
```
```output
[Jan 10, 10:53:28] Loading segmented_maxsim_cpp extension (set COLBERT_LOAD_TORCH_EXTENSION_VERBOSE=True for more info)...
``````output
/Users/harrisonchase/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/torch/cuda/amp/grad_scaler.py:125: UserWarning: torch.cuda.amp.GradScaler is enabled, but CUDA is not available.  Disabling.
  warnings.warn(
```

## Retriever

我们可以使用 RAGatouille 作为检索器。有关更多信息，请参见 [RAGatouille Retriever](/docs/integrations/retrievers/ragatouille)

## 文档压缩器

我们还可以将 RAGatouille 作为现成的重排序器使用。这将允许我们使用 ColBERT 对来自任何通用检索器的检索结果进行重排序。这样做的好处是我们可以在任何现有索引的基础上进行操作，因此我们不需要创建新的索引。我们可以通过使用 LangChain 中的 [document compressor](/docs/how_to/contextual_compression) 抽象来实现这一点。

## 设置基础检索器

首先，让我们以一个示例来设置一个基础检索器。


```python
import requests
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter


def get_wikipedia_page(title: str):
    """
    检索维基百科页面的完整文本内容。

    :param title: str - 维基百科页面的标题。
    :return: str - 页面完整文本内容，作为原始字符串。
    """
    # 维基百科 API 端点
    URL = "https://en.wikipedia.org/w/api.php"

    # API 请求的参数
    params = {
        "action": "query",
        "format": "json",
        "titles": title,
        "prop": "extracts",
        "explaintext": True,
    }

    # 自定义 User-Agent 头以遵守维基百科的最佳实践
    headers = {"User-Agent": "RAGatouille_tutorial/0.0.1 (ben@clavie.eu)"}

    response = requests.get(URL, params=params, headers=headers)
    data = response.json()

    # 提取页面内容
    page = next(iter(data["query"]["pages"].values()))
    return page["extract"] if "extract" in page else None


text = get_wikipedia_page("Hayao_Miyazaki")
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=0)
texts = text_splitter.create_documents([text])
```


```python
retriever = FAISS.from_documents(texts, OpenAIEmbeddings()).as_retriever(
    search_kwargs={"k": 10}
)
```


```python
docs = retriever.invoke("Miyazaki 创立了哪个动画工作室")
docs[0]
```



```output
Document(page_content='collaborative projects. In April 1984, Miyazaki opened his own office in Suginami Ward, naming it Nibariki.')
```


我们可以看到结果与所问的问题并不是非常相关。

## 使用 ColBERT 作为重排序器


```python
from langchain.retrievers import ContextualCompressionRetriever

compression_retriever = ContextualCompressionRetriever(
    base_compressor=RAG.as_langchain_document_compressor(), base_retriever=retriever
)

compressed_docs = compression_retriever.invoke(
    "宫崎骏创办了哪个动画工作室"
)
```
```output
/Users/harrisonchase/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/torch/amp/autocast_mode.py:250: UserWarning: User provided device_type of 'cuda', but CUDA is not available. Disabling
  warnings.warn(
```

```python
compressed_docs[0]
```



```output
Document(page_content='在1985年6月，宫崎骏、高畑勋、德间书店和铃木敏夫共同创办了动画制作公司吉卜力工作室，资金来自德间书店。吉卜力工作室的第一部电影《天空之城》（1986）使用了《风之谷》的同一制作团队。宫崎骏为电影设置的设计灵感来自希腊建筑和“欧洲城市模板”。电影中的一些建筑也受到威尔士一个矿业小镇的启发；宫崎骏在第一次目睹矿工罢工时', metadata={'relevance_score': 26.5194149017334})
```


这个答案相关性更高！