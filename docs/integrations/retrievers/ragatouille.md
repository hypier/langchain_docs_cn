---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/ragatouille.ipynb
---

# RAGatouille

>[RAGatouille](https://github.com/bclavie/RAGatouille) 让使用 `ColBERT` 变得简单易行！
>
>[ColBERT](https://github.com/stanford-futuredata/ColBERT) 是一个快速且准确的检索模型，能够在数十毫秒内对大文本集合进行可扩展的基于 BERT 的搜索。

我们可以将其用作 [retriever](/docs/how_to#retrievers)。它将展示该集成特定的功能。浏览完后，探索 [相关用例页面](/docs/how_to#qa-with-rag) 可能会有帮助，以了解如何将此向量存储作为更大链的一部分使用。

本页面涵盖如何在 LangChain 链中使用 [RAGatouille](https://github.com/bclavie/RAGatouille) 作为检索器。

## 设置

集成位于 `ragatouille` 包中。

```bash
pip install -U ragatouille
```

## 用法

这个示例取自他们的文档


```python
from ragatouille import RAGPretrainedModel

RAG = RAGPretrainedModel.from_pretrained("colbert-ir/colbertv2.0")
```


```python
import requests


def get_wikipedia_page(title: str):
    """
    检索维基百科页面的完整文本内容。

    :param title: str - 维基百科页面的标题。
    :return: str - 页面完整文本内容的原始字符串。
    """
    # 维基百科API端点
    URL = "https://en.wikipedia.org/w/api.php"

    # API请求的参数
    params = {
        "action": "query",
        "format": "json",
        "titles": title,
        "prop": "extracts",
        "explaintext": True,
    }

    # 自定义User-Agent头以遵循维基百科的最佳实践
    headers = {"User-Agent": "RAGatouille_tutorial/0.0.1 (ben@clavie.eu)"}

    response = requests.get(URL, params=params, headers=headers)
    data = response.json()

    # 提取页面内容
    page = next(iter(data["query"]["pages"].values()))
    return page["extract"] if "extract" in page else None
```


```python
full_document = get_wikipedia_page("Hayao_Miyazaki")
```


```python
RAG.index(
    collection=[full_document],
    index_name="Miyazaki-123",
    max_document_length=180,
    split_documents=True,
)
```
```output


[Jan 07, 10:38:18] #> 创建目录 .ragatouille/colbert/indexes/Miyazaki-123 


#> 启动中...
[Jan 07, 10:38:23] 加载 segmented_maxsim_cpp 扩展 (设置 COLBERT_LOAD_TORCH_EXTENSION_VERBOSE=True 以获取更多信息)...
``````output
/Users/harrisonchase/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/torch/cuda/amp/grad_scaler.py:125: UserWarning: torch.cuda.amp.GradScaler 已启用，但 CUDA 不可用。  禁用中。
  warnings.warn(

  0%|          | 0/2 [00:00<?, ?it/s]/Users/harrisonchase/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/torch/amp/autocast_mode.py:250: UserWarning: 用户提供的 device_type 为 'cuda'，但 CUDA 不可用。 禁用中
  warnings.warn(
``````output
[Jan 07, 10:38:24] [0] 		 #> 编码 81 段落..
``````output
 50%|█████     | 1/2 [00:02<00:02,  2.85s/it]/Users/harrisonchase/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/torch/amp/autocast_mode.py:250: UserWarning: 用户提供的 device_type 为 'cuda'，但 CUDA 不可用。 禁用中
  warnings.warn(
100%|██████████| 2/2 [00:03<00:00,  1.74s/it]
WARNING clustering 10001 points to 1024 centroids: please provide at least 39936 training points
``````output
[Jan 07, 10:38:27] [0] 		 avg_doclen_est = 129.9629669189453 	 len(local_sample) = 81
[Jan 07, 10:38:27] [0] 		 创建 1,024 个分区。
[Jan 07, 10:38:27] [0] 		 *估计* 10,527 个嵌入。
[Jan 07, 10:38:27] [0] 		 #> 保存索引计划到 .ragatouille/colbert/indexes/Miyazaki-123/plan.json ..
对 128D 中的 10001 个点进行聚类到 1024 个簇，重做 1 次，20 次迭代
  预处理时间为 0.00 秒
  迭代 0 (0.02 秒，搜索 0.02 秒): 目标=3772.41 不平衡=1.562 nsplit=0       
  迭代 1 (0.02 秒，搜索 0.02 秒): 目标=2408.99 不平衡=1.470 nsplit=1       
  迭代 2 (0.03 秒，搜索 0.03 秒): 目标=2243.87 不平衡=1.450 nsplit=0       
  迭代 3 (0.04 秒，搜索 0.04 秒): 目标=2168.48 不平衡=1.444 nsplit=0       
  迭代 4 (0.05 秒，搜索 0.05 秒): 目标=2134.26 不平衡=1.449 nsplit=0       
  迭代 5 (0.06 秒，搜索 0.05 秒): 目标=2117.18 不平衡=1.449 nsplit=0       
  迭代 6 (0.06 秒，搜索 0.06 秒): 目标=2108.43 不平衡=1.449 nsplit=0       
  迭代 7 (0.07 秒，搜索 0.07 秒): 目标=2102.62 不平衡=1.450 nsplit=0       
  迭代 8 (0.08 秒，搜索 0.08 秒): 目标=2100.68 不平衡=1.451 nsplit=0       
  迭代 9 (0.09 秒，搜索 0.08 秒): 目标=2099.66 不平衡=1.451 nsplit=0       
  迭代 10 (0.10 秒，搜索 0.09 秒): 目标=2099.03 不平衡=1.451 nsplit=0       
  迭代 11 (0.10 秒，搜索 0.10 秒): 目标=2098.67 不平衡=1.453 nsplit=0       
  迭代 12 (0.11 秒，搜索 0.11 秒): 目标=2097.78 不平衡=1.455 nsplit=0       
  迭代 13 (0.12 秒，搜索 0.12 秒): 目标=2097.31 不平衡=1.455 nsplit=0       
  迭代 14 (0.13 秒，搜索 0.12 秒): 目标=2097.13 不平衡=1.455 nsplit=0       
  迭代 15 (0.14 秒，搜索 0.13 秒): 目标=2097.09 不平衡=1.455 nsplit=0       
  迭代 16 (0.14 秒，搜索 0.14 秒): 目标=2097.09 不平衡=1.455 nsplit=0       
  迭代 17 (0.15 秒，搜索 0.15 秒): 目标=2097.09 不平衡=1.455 nsplit=0       
  迭代 18 (0.16 秒，搜索 0.15 秒): 目标=2097.09 不平衡=1.455 nsplit=0       
  迭代 19 (0.17 秒，搜索 0.16 秒): 目标=2097.09 不平衡=1.455 nsplit=0       
[0.037, 0.038, 0.041, 0.036, 0.035, 0.036, 0.034, 0.036, 0.034, 0.034, 0.036, 0.037, 0.032, 0.039, 0.035, 0.039, 0.033, 0.035, 0.035, 0.037, 0.037, 0.037, 0.037, 0.037, 0.038, 0.034, 0.037, 0.035, 0.036, 0.037, 0.036, 0.04, 0.037, 0.037, 0.036, 0.034, 0.037, 0.035, 0.038, 0.039, 0.037, 0.039, 0.035, 0.036, 0.036, 0.035, 0.035, 0.038, 0.037, 0.033, 0.036, 0.032, 0.034, 0.035, 0.037, 0.037, 0.041, 0.037, 0.039, 0.033, 0.035, 0.033, 0.036, 0.036, 0.038, 0.036, 0.037, 0.038, 0.035, 0.035, 0.033, 0.034, 0.032, 0.038, 0.037, 0.037, 0.036, 0.04, 0.033, 0.037, 0.035, 0.04, 0.036, 0.04, 0.032, 0.037, 0.036, 0.037, 0.034, 0.042, 0.037, 0.035, 0.035, 0.038, 0.034, 0.036, 0.041, 0.035, 0.036, 0.037, 0.041, 0.04, 0.036, 0.036, 0.035, 0.036, 0.034, 0.033, 0.036, 0.033, 0.037, 0.038, 0.036, 0.033, 0.038, 0.037, 0.038, 0.037, 0.039, 0.04, 0.034, 0.034, 0.036, 0.039, 0.033, 0.037, 0.035, 0.037]
[Jan 07, 10:38:27] [0] 		 #> 编码 81 段落..
``````output
0it [00:00, ?it/s]
  0%|          | 0/2 [00:00<?, ?it/s][A
 50%|█████     | 1/2 [00:02<00:02,  2.53s/it][A
100%|██████████| 2/2 [00:03<00:00,  1.56s/it][A
1it [00:03,  3.16s/it]
100%|██████████| 1/1 [00:00<00:00, 4017.53it/s]
100%|██████████| 1024/1024 [00:00<00:00, 306105.57it/s]
``````output
[Jan 07, 10:38:30] #> 优化 IVF 以存储从质心到 PID 列表的映射..
[Jan 07, 10:38:30] #> 构建 emb2pid 映射..
[Jan 07, 10:38:30] len(emb2pid) = 10527
[Jan 07, 10:38:30] #> 保存优化后的 IVF 到 .ragatouille/colbert/indexes/Miyazaki-123/ivf.pid.pt

#> 合并中...
完成索引！
```

```python
results = RAG.search(query="宫崎骏创办了哪个动画工作室？", k=3)
```
```output
第一次加载索引 Miyazaki-123 的搜索器... 这可能需要几秒钟
[Jan 07, 10:38:34] 加载 segmented_maxsim_cpp 扩展 (设置 COLBERT_LOAD_TORCH_EXTENSION_VERBOSE=True 以获取更多信息)...
[Jan 07, 10:38:35] #> 加载编解码器...
[Jan 07, 10:38:35] #> 加载 IVF...
[Jan 07, 10:38:35] 加载 segmented_lookup_cpp 扩展 (设置 COLBERT_LOAD_TORCH_EXTENSION_VERBOSE=True 以获取更多信息)...
``````output
/Users/harrisonchase/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/torch/cuda/amp/grad_scaler.py:125: UserWarning: torch.cuda.amp.GradScaler 已启用，但 CUDA 不可用。  禁用中。
  warnings.warn(
``````output
[Jan 07, 10:38:35] #> 加载文档长度...
``````output
100%|███████████████████████████████████████████████████████| 1/1 [00:00<00:00, 3872.86it/s]
``````output
[Jan 07, 10:38:35] #> 加载代码和残差...
``````output

100%|████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 604.89it/s]
``````output
[Jan 07, 10:38:35] 加载 filter_pids_cpp 扩展 (设置 COLBERT_LOAD_TORCH_EXTENSION_VERBOSE=True 以获取更多信息)...
``````output

``````output
[Jan 07, 10:38:35] 加载 decompress_residuals_cpp 扩展 (设置 COLBERT_LOAD_TORCH_EXTENSION_VERBOSE=True 以获取更多信息)...
搜索器加载完成！

#> QueryTokenizer.tensorize(batch_text[0], batch_background[0], bsize) ==
#> 输入: . 宫崎骏创办了哪个动画工作室？， 		 True, 		 None
#> 输出 ID: torch.Size([32]), tensor([  101,     1,  2054,  7284,  2996,  2106,  2771,  3148, 18637,  2179,
         1029,   102,   103,   103,   103,   103,   103,   103,   103,   103,
          103,   103,   103,   103,   103,   103,   103,   103,   103,   103,
          103,   103])
#> 输出掩码: torch.Size([32]), tensor([1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
        0, 0, 0, 0, 0, 0, 0, 0])
``````output
/Users/harrisonchase/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/torch/amp/autocast_mode.py:250: UserWarning: 用户提供的 device_type 为 'cuda'，但 CUDA 不可用。 禁用中
  warnings.warn(
```

```python
results
```



```output
[{'content': '在1984年4月，宫崎骏在杉并区开设了自己的工作室，命名为 Nibariki。\n\n\n=== 吉卜力工作室 ===\n\n\n==== 早期电影 (1985–1996) ====\n在1985年6月，宫崎骏、高畑勋、德间书店和铃木敏夫共同创办了动画制作公司吉卜力工作室，资金来自德间书店。吉卜力工作室的第一部电影《天空之城》（1986）使用了与《风之谷》相同的制作团队。宫崎骏为影片设置的设计灵感来自希腊建筑和“欧洲城市模板”。',
  'score': 25.90749740600586,
  'rank': 1},
 {'content': '宫崎骏（宮崎 駿 或 宮﨑 駿，Miyazaki Hayao，[mijaꜜzaki hajao]；生于1941年1月5日）是一位日本动画师、电影制片人和漫画艺术家。作为吉卜力工作室的共同创始人，他作为一位杰出的讲故事者和日本动画长片的创作者获得了国际赞誉，并被广泛认为是动画史上最成功的电影制片人之一。\n宫崎骏出生于日本帝国的东京市，从小对漫画和动画表现出兴趣，并于1963年加入东映动画。在东映动画的早期，他担任了中间动画师，并与导演高畑勋合作。',
  'score': 25.4748477935791,
  'rank': 2},
 {'content': '格伦·基恩表示，宫崎骏对华特迪士尼动画工作室影响巨大，自《拯救大兵瑞恩》（1990）以来，他一直是“我们传统的一部分”。迪士尼文艺复兴时代的到来也受到宫崎骏电影发展的影响。皮克斯和阿德曼工作室的艺术家们签署了一份致敬，表示：“您是我们的灵感，宫崎骏！”',
  'score': 24.84897232055664,
  'rank': 3}]
```

我们可以轻松地转换为 LangChain 检索器！在创建时，我们可以传入任何想要的 kwargs（例如 `k`）

```python
retriever = RAG.as_langchain_retriever(k=3)
```

```python
retriever.invoke("What animation studio did Miyazaki found?")
```
```output
/Users/harrisonchase/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/torch/amp/autocast_mode.py:250: UserWarning: User provided device_type of 'cuda', but CUDA is not available. Disabling
  warnings.warn(
```

```output
[Document(page_content='In April 1984, Miyazaki opened his own office in Suginami Ward, naming it Nibariki.\n\n\n=== Studio Ghibli ===\n\n\n==== Early films (1985–1996) ====\nIn June 1985, Miyazaki, Takahata, Tokuma and Suzuki founded the animation production company Studio Ghibli, with funding from Tokuma Shoten. Studio Ghibli\'s first film, Laputa: Castle in the Sky (1986), employed the same production crew of Nausicaä. Miyazaki\'s designs for the film\'s setting were inspired by Greek architecture and "European urbanistic templates".'),
 Document(page_content='Hayao Miyazaki (宮崎 駿 or 宮﨑 駿, Miyazaki Hayao, [mijaꜜzaki hajao]; born January 5, 1941) is a Japanese animator, filmmaker, and manga artist. A co-founder of Studio Ghibli, he has attained international acclaim as a masterful storyteller and creator of Japanese animated feature films, and is widely regarded as one of the most accomplished filmmakers in the history of animation.\nBorn in Tokyo City in the Empire of Japan, Miyazaki expressed interest in manga and animation from an early age, and he joined Toei Animation in 1963. During his early years at Toei Animation he worked as an in-between artist and later collaborated with director Isao Takahata.'),
 Document(page_content='Glen Keane said Miyazaki is a "huge influence" on Walt Disney Animation Studios and has been "part of our heritage" ever since The Rescuers Down Under (1990). The Disney Renaissance era was also prompted by competition with the development of Miyazaki\'s films. Artists from Pixar and Aardman Studios signed a tribute stating, "You\'re our inspiration, Miyazaki-san!"')]
```

## 链接

我们可以轻松地将这个检索器组合成一个链。

```python
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_template(
    """Answer the following question based only on the provided context:

<context>
{context}
</context>

Question: {input}"""
)

llm = ChatOpenAI()

document_chain = create_stuff_documents_chain(llm, prompt)
retrieval_chain = create_retrieval_chain(retriever, document_chain)
```

```python
retrieval_chain.invoke({"input": "What animation studio did Miyazaki found?"})
```
```output
/Users/harrisonchase/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/torch/amp/autocast_mode.py:250: UserWarning: User provided device_type of 'cuda', but CUDA is not available. Disabling
  warnings.warn(
```

```output
{'input': 'What animation studio did Miyazaki found?',
 'context': [Document(page_content='In April 1984, Miyazaki opened his own office in Suginami Ward, naming it Nibariki.\n\n\n=== Studio Ghibli ===\n\n\n==== Early films (1985–1996) ====\nIn June 1985, Miyazaki, Takahata, Tokuma and Suzuki founded the animation production company Studio Ghibli, with funding from Tokuma Shoten. Studio Ghibli\'s first film, Laputa: Castle in the Sky (1986), employed the same production crew of Nausicaä. Miyazaki\'s designs for the film\'s setting were inspired by Greek architecture and "European urbanistic templates".'),
  Document(page_content='Hayao Miyazaki (宮崎 駿 or 宮﨑 駿, Miyazaki Hayao, [mijaꜜzaki hajao]; born January 5, 1941) is a Japanese animator, filmmaker, and manga artist. A co-founder of Studio Ghibli, he has attained international acclaim as a masterful storyteller and creator of Japanese animated feature films, and is widely regarded as one of the most accomplished filmmakers in the history of animation.\nBorn in Tokyo City in the Empire of Japan, Miyazaki expressed interest in manga and animation from an early age, and he joined Toei Animation in 1963. During his early years at Toei Animation he worked as an in-between artist and later collaborated with director Isao Takahata.'),
  Document(page_content='Glen Keane said Miyazaki is a "huge influence" on Walt Disney Animation Studios and has been "part of our heritage" ever since The Rescuers Down Under (1990). The Disney Renaissance era was also prompted by competition with the development of Miyazaki\'s films. Artists from Pixar and Aardman Studios signed a tribute stating, "You\'re our inspiration, Miyazaki-san!"')],
 'answer': 'Miyazaki founded Studio Ghibli.'}
```

```python
for s in retrieval_chain.stream({"input": "What animation studio did Miyazaki found?"}):
    print(s.get("answer", ""), end="")
```
```output
/Users/harrisonchase/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/torch/amp/autocast_mode.py:250: UserWarning: User provided device_type of 'cuda', but CUDA is not available. Disabling
  warnings.warn(
``````output
Miyazaki founded Studio Ghibli.
```

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)