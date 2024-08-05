---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/news.ipynb
---

# 新闻 URL

这部分介绍了如何从 URL 列表加载 HTML 新闻文章，并将其转换为我们可以在后续处理中使用的文档格式。

```python
from langchain_community.document_loaders import NewsURLLoader
```

```python
urls = [
    "https://www.bbc.com/news/world-us-canada-66388172",
    "https://www.bbc.com/news/entertainment-arts-66384971",
]
```

将 urls 传入以加载到文档中

```python
loader = NewsURLLoader(urls=urls)
data = loader.load()
print("第一篇文章: ", data[0])
print("\n第二篇文章: ", data[1])
```
```output
第一篇文章:  page_content='在对审查1月6日骚乱的国会委员会作证时，鲍威尔女士表示，她并未审查自己所做的众多选举舞弊指控，告诉他们“没有理智的人”会将她的指控视为事实。她和她的代表均未发表评论。' metadata={'title': '唐纳德·特朗普起诉：我们对六名共谋者了解多少？', 'link': 'https://www.bbc.com/news/world-us-canada-66388172', 'authors': [], 'language': 'en', 'description': '检察官描述了六名被指控帮助特朗普破坏选举的人。', 'publish_date': None}

第二篇文章:  page_content='威廉姆斯女士补充道：“如果我能做任何事情来确保与她合作的舞者或歌手不必经历同样的经历，我会这样做。”' metadata={'title': "莉佐舞者阿里安娜·戴维斯和克里斯托尔·威廉姆斯：'没有人敢发声，他们很害怕'", 'link': 'https://www.bbc.com/news/entertainment-arts-66384971', 'authors': [], 'language': 'en', 'description': '这位美国流行歌手因性骚扰和体重羞辱而被起诉，但尚未发表评论。', 'publish_date': None}
```
使用 nlp=True 进行 nlp 分析并生成关键词和摘要

```python
loader = NewsURLLoader(urls=urls, nlp=True)
data = loader.load()
print("第一篇文章: ", data[0])
print("\n第二篇文章: ", data[1])
```
```output
第一篇文章:  page_content='在对审查1月6日骚乱的国会委员会作证时，鲍威尔女士表示，她并未审查自己所做的众多选举舞弊指控，告诉他们“没有理智的人”会将她的指控视为事实。她和她的代表均未发表评论。' metadata={'title': '唐纳德·特朗普起诉：我们对六名共谋者了解多少？', 'link': 'https://www.bbc.com/news/world-us-canada-66388172', 'authors': [], 'language': 'en', 'description': '检察官描述了六名被指控帮助特朗普破坏选举的人。', 'publish_date': None, 'keywords': ['powell', 'know', 'donald', 'trump', 'review', 'indictment', 'telling', 'view', 'reasonable', 'person', 'testimony', 'coconspirators', 'riot', 'representatives', 'claims'], 'summary': '在对审查1月6日骚乱的国会委员会作证时，鲍威尔女士表示，她并未审查自己所做的众多选举舞弊指控，告诉他们“没有理智的人”会将她的指控视为事实。\n她和她的代表均未发表评论。'}

第二篇文章:  page_content='威廉姆斯女士补充道：“如果我能做任何事情来确保与她合作的舞者或歌手不必经历同样的经历，我会这样做。”' metadata={'title': "莉佐舞者阿里安娜·戴维斯和克里斯托尔·威廉姆斯：'没有人敢发声，他们很害怕'", 'link': 'https://www.bbc.com/news/entertainment-arts-66384971', 'authors': [], 'language': 'en', 'description': '这位美国流行歌手因性骚扰和体重羞辱而被起诉，但尚未发表评论。', 'publish_date': None, 'keywords': ['davis', 'lizzo', 'singers', 'experience', 'crystal', 'ensure', 'arianna', 'theres', 'williams', 'power', 'going', 'dancers', 'im', 'speaks', 'work', 'ms', 'scared'], 'summary': '威廉姆斯女士补充道：“如果我能做任何事情来确保与她合作的舞者或歌手不必经历同样的经历，我会这样做。”'}
```

```python
data[0].metadata["keywords"]
```

```output
['powell',
 'know',
 'donald',
 'trump',
 'review',
 'indictment',
 'telling',
 'view',
 'reasonable',
 'person',
 'testimony',
 'coconspirators',
 'riot',
 'representatives',
 'claims']
```

```python
data[0].metadata["summary"]
```

```output
'在对审查1月6日骚乱的国会委员会作证时，鲍威尔女士表示，她并未审查自己所做的众多选举舞弊指控，告诉他们“没有理智的人”会将她的指控视为事实。\n她和她的代表均未发表评论。'
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)