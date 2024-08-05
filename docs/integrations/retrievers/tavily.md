---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/tavily.ipynb
sidebar_label: TavilySearchAPI
---

# TavilySearchAPIRetriever

## 概述
>[Tavily的搜索API](https://tavily.com) 是一个专为AI代理（LLMs）构建的搜索引擎，能够快速提供实时、准确和事实性的结果。

我们可以将其用作[检索器](/docs/how_to#retrievers)。它将展示与此集成相关的功能。浏览完后，探索[相关用例页面](/docs/how_to#qa-with-rag)可能会对学习如何将此向量存储作为更大链的一部分使用有所帮助。

### 集成细节

| 检索器 | 来源 | 包 |
| :--- | :--- | :---: |
[TavilySearchAPIRetriever](https://api.python.langchain.com/en/latest/retrievers/langchain_community.retrievers.tavily_search_api.TavilySearchAPIRetriever.html) | 互联网搜索 | langchain_community |

## 设置

如果您想从单个查询获取自动化追踪，您还可以通过取消注释以下内容来设置您的 [LangSmith](https://docs.smith.langchain.com/) API 密钥：

```python
# os.environ["LANGSMITH_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
# os.environ["LANGSMITH_TRACING"] = "true"
```

### 安装

集成位于 `langchain-community` 包中。我们还需要安装 `tavily-python` 包本身。


```python
%pip install -qU langchain-community tavily-python
```

我们还需要设置我们的 Tavily API 密钥。


```python
import getpass
import os

os.environ["TAVILY_API_KEY"] = getpass.getpass()
```

## 实例化

现在我们可以实例化我们的检索器：


```python
from langchain_community.retrievers import TavilySearchAPIRetriever

retriever = TavilySearchAPIRetriever(k=3)
```

## 用法


```python
query = "what year was breath of the wild released?"

retriever.invoke(query)
```



```output
[Document(metadata={'title': '塞尔达传说：荒野之息 - Nintendo Switch Wiki', 'source': 'https://nintendo-switch.fandom.com/wiki/The_Legend_of_Zelda:_Breath_of_the_Wild', 'score': 0.9961155, 'images': []}, page_content='塞尔达传说：荒野之息是一款由任天堂为Wii U发布的开放世界动作冒险游戏，并作为Nintendo Switch的首发游戏，于2017年3月3日在全球发布。它是塞尔达传说系列的第十九部作品，也是首个以HD分辨率开发的作品。该游戏拥有一个巨大的开放世界，玩家可以...'),
 Document(metadata={'title': '塞尔达传说：荒野之息 - Zelda Wiki', 'source': 'https://zelda.fandom.com/wiki/The_Legend_of_Zelda:_Breath_of_the_Wild', 'score': 0.9804313, 'images': []}, page_content='[]\n参考资料\n塞尔达传说\xa0·\n林克的冒险\xa0·\n时之笛 (& 四剑) \xa0·\n林克的觉醒 (DX; Nintendo Switch) \xa0·\n时之笛 (大师之旅; 3D) \xa0·\n穆杰拉的面具 (3D) \xa0·\n时代的神谕 \xa0·\n季节的神谕 \xa0·\n四剑 (周年版) \xa0·\n风之杖 (HD) \xa0·\n四剑冒险 \xa0·\n迷你帽 \xa0·\n黄昏公主 (HD) \xa0·\n幽灵时钟 \xa0·\n灵轨 \xa0·\n天空之剑 (HD) \xa0·\n两个世界的连结 \xa0·\n三角力量英雄 \xa0·\n荒野之息 \xa0·\n王国之泪\n塞尔达 (Game & Watch) \xa0·\n塞尔达传说 Game Watch \xa0·\n林克的弩训练 \xa0·\n我的任天堂拼图：黄昏公主 \xa0·\n海拉鲁的节奏 \xa0·\nGame & Watch: 塞尔达传说\nCD-i游戏\n 列表[]\n角色[]\nBoss[]\n敌人[]\n地下城[]\n地点[]\n道具[]\n翻译[]\n致谢[]\n评价[]\n销量[]\n青沼英二和藤林秀麻在2017年游戏大奖上为《荒野之息》接受“年度游戏”奖\n荒野之息在前两周的销量估计约为130万份，约89%的Switch用户估计也购买了该游戏。[52] 游戏的销量一直保持强劲，截至2022年6月30日，Switch版在全球销量达到2714万份，而Wii U版截至2019年12月31日销量为169万份，[53][54] 使《荒野之息》的总销量达到2883万份。\n 它还获得了来自100多位评论家的Metacritic评分97，成为有史以来评分最高的游戏之一。[59][60] 值得注意的是，该游戏在Metacritic上获得的完美评分是截至那时为止任何游戏中最多的。[61]\n在2022年，《荒野之息》被选为有史以来最佳的塞尔达传说游戏，在他们的“十大最佳塞尔达游戏”列表倒计时中；但在2023年，他们的新版本“十大最佳塞尔达游戏”列表中将其排为“第二”最佳塞尔达游戏，仅次于其续作，视频游戏经典评选将《荒野之息》列为有史以来最佳视频游戏之一。[74] Metacritic将《荒野之息》评为2010年代最佳游戏。[75]\n粉丝评价[]\nWatchMojo将《荒野之息》排在他们“有史以来十大塞尔达传说游戏”列表的第2位，仅次于时之笛。[76] 邪恶的面孔 \xa0·\n甘美隆之杖 \xa0·\n塞尔达的冒险\n海拉鲁战争系列\n海拉鲁战争 (传奇; 完整版) \xa0·\n海拉鲁战争：灾厄时代\n卫星视图游戏\nBS 塞尔达传说 \xa0·\n古代石碑\n丁格系列\n新鲜采摘的丁格的玫瑰鲁比兰 \xa0·\n丁格的气球战斗DS \xa0·\n'),
 Document(metadata={'title': '塞尔达传说：荒野之息 - Zelda Wiki', 'source': 'https://zeldawiki.wiki/wiki/The_Legend_of_Zelda:_Breath_of_the_Wild', 'score': 0.9627432, 'images': []}, page_content='塞尔达传说\xa0•\n林克的冒险\xa0•\n时之笛 (& 四剑)\xa0•\n林克的觉醒 (DX; Nintendo Switch)\xa0•\n时之笛 (大师之旅; 3D)\xa0•\n穆杰拉的面具 (3D)\xa0•\n时代的神谕\xa0•\n季节的神谕\xa0•\n四剑 (周年版)\xa0•\n风之杖 (HD)\xa0•\n四剑冒险\xa0•\n迷你帽\xa0•\n黄昏公主 (HD)\xa0•\n幽灵时钟\xa0•\n灵轨\xa0•\n天空之剑 (HD)\xa0•\n两个世界的连结\xa0•\n三角力量英雄\xa0•\n荒野之息\xa0•\n王国之泪\n塞尔达 (Game & Watch)\xa0•\n塞尔达传说 Game Watch\xa0•\n海拉鲁英雄\xa0•\n林克的弩训练\xa0•\n我的任天堂拼图：黄昏公主\xa0•\n海拉鲁的节奏\xa0•\n害虫\n邪恶的面孔\xa0•\n甘美隆之杖\xa0•\n塞尔达的冒险\n海拉鲁战争 (传奇; 完整版)\xa0•\n海拉鲁战争：灾厄时代\nBS 塞尔达传说\xa0•\n古代石碑\n新鲜采摘的丁格的玫瑰鲁比兰\xa0•\n丁格的气球战斗DS\xa0•\n丁格的过多包\xa0•\n成熟丁格的爱情气球之旅\n魂之刃II\xa0•\n瓦里奥制造系列\xa0•\n彩虹船长\xa0•\n任天堂乐园\xa0•\n拼字无限\xa0•\n马里奥卡丁车8\xa0•\n喷射战士3\n超级粉碎兄弟 (系列)\n超级粉碎兄弟\xa0•\n超级粉碎兄弟：梅利\xa0•\n超级粉碎兄弟：斗士\xa0•\n超级粉碎兄弟：任天堂3DS / Wii U\xa0•\n 它还获得了来自100多位评论家的Metacritic评分97，成为有史以来评分最高的游戏之一。[60][61] 值得注意的是，该游戏在Metacritic上获得的完美评分是截至那时为止任何游戏中最多的。[62]\n奖项\n在2016年，荒野之息作为一款备受期待的游戏获得了多个奖项，包括IGN和Destructoid的E3最佳游戏，[63][64] 在2016年游戏评论奖，[65] 和2016年游戏大奖。[66] 发布后，荒野之息获得了2017年日本游戏大奖的“年度游戏”称号，[67] 2017年金摇杆奖,<ref我们的最终奖项是年度最佳游戏。官方网站\n官方网站\n正典性\n正典性\n正典[citation needed]\n前作\n前作\n三角力量英雄\n续作\n续作\n王国之泪\n塞尔达传说：荒野之息指南在StrategyWiki\n荒野之息指南在Zelda Universe\n塞尔达传说：荒野之息是塞尔达传说系列的第十九部主要作品。列表\n角色\nBoss\n敌人\n地下城\n地点\n道具\n翻译\n致谢\n评价\n销量\n荒野之息在前两周的销量估计约为130万份，约89%的Switch用户估计也购买了该游戏。[53] 游戏的销量一直保持强劲，截至2023年9月30日，Switch版在全球销量达到3115万份，而Wii U版截至2021年12月31日销量为170万份，[54][55] 使《荒野之息》的总销量达到3285万份。\n 塞尔达传说：荒野之息\n塞尔达传说：荒野之息\n塞尔达传说：荒野之息\n开发者\n开发者\n发行商\n发行商\n任天堂\n设计师\n设计师\n')]
```

## 在链中使用

我们可以轻松地将这个检索器组合到一个链中。


```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_template(
    """Answer the question based only on the context provided.

Context: {context}

Question: {question}"""
)

llm = ChatOpenAI(model="gpt-3.5-turbo-0125")


def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)


chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)
```


```python
chain.invoke("how many units did bretch of the wild sell in 2020")
```



```output
'As of August 2020, The Legend of Zelda: Breath of the Wild had sold over 20.1 million copies worldwide on Nintendo Switch and Wii U.'
```

## API 参考

有关所有 `TavilySearchAPIRetriever` 功能和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/retrievers/langchain_community.retrievers.tavily_search_api.TavilySearchAPIRetriever.html)。

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)