---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/rss.ipynb
---

# RSS Feeds

这部分介绍了如何从一系列 RSS 源 URL 加载 HTML 新闻文章到我们可以下游使用的文档格式中。


```python
%pip install --upgrade --quiet  feedparser newspaper3k listparser
```


```python
from langchain_community.document_loaders import RSSFeedLoader
```


```python
urls = ["https://news.ycombinator.com/rss"]
```

将 urls 传入以加载它们到文档中


```python
loader = RSSFeedLoader(urls=urls)
data = loader.load()
print(len(data))
```


```python
print(data[0].page_content)
```
```output
(next Rich)

04 August 2023

Rich Hickey

我怀着一份心痛和乐观的心情，今天宣布我（早已计划好的）从商业软件开发和在 Nubank 的工作中退休。看到 Clojure 和 Datomic 成功地大规模应用，令人振奋。

我期待着继续与 Alex、Stu、Fogus 和许多其他人一起，作为独立开发者，领导持续的工作，维护和增强 Clojure。我们为 1.12 及以后的版本计划了许多有用的东西。社区依然友好、成熟且富有成效，并正在将 Clojure 带入许多有趣的新领域。

我想特别感谢 Nubank 对 Alex、Fogus 和核心团队的持续赞助，以及对整个 Clojure 社区的支持。

Stu 将继续在 Nubank 领导 Datomic 的开发，Datomic 团队在不断壮大和蓬勃发展。我特别期待新免费可用的 Datomic 将引领我们走向何方。

我在 Cognitect 的时光仍然是我职业生涯的亮点。我从我们团队的每一个人身上都学到了很多，永远感激大家的互动。这里有太多的人值得感谢，但我必须向 Stu 和 Justin 表达我最诚挚的感谢和爱，感谢他们（一次又一次）对我和我的想法冒险，并始终是最好的合作伙伴和朋友，完全体现了诚信的理念。当然，还有 Alex Miller - 他拥有我所缺乏的许多技能，如果没有他那不可动摇的精神、积极性和友谊，Clojure 不会成为今天的样子。

我通过 Clojure 和 Cognitect 结交了许多朋友，希望在未来能够继续培养这些友谊。

退休让我回到了最初开发 Clojure 时的自由和独立。旅程仍在继续！
```
您可以向 NewsURLLoader 传递参数，它将用于加载文章。


```python
loader = RSSFeedLoader(urls=urls, nlp=True)
data = loader.load()
print(len(data))
```
```output
Error fetching or processing https://twitter.com/andrewmccalip/status/1687405505604734978, exception: You must `parse()` an article first!
Error processing entry https://twitter.com/andrewmccalip/status/1687405505604734978, exception: list index out of range
``````output
13
```

```python
data[0].metadata["keywords"]
```



```output
['nubank',
 'alex',
 'stu',
 'taking',
 'team',
 'remains',
 'rich',
 'clojure',
 'thank',
 'planned',
 'datomic']
```



```python
data[0].metadata["summary"]
```



```output
'看到 Clojure 和 Datomic 成功地大规模应用，令人振奋。\n我期待着继续与 Alex、Stu、Fogus 和许多其他人一起，作为独立开发者，领导持续的工作，维护和增强 Clojure。\n社区依然友好、成熟且富有成效，并正在将 Clojure 带入许多有趣的新领域。\n我想特别感谢 Nubank 对 Alex、Fogus 和核心团队的持续赞助，以及对整个 Clojure 社区的支持。\nStu 将继续在 Nubank 领导 Datomic 的开发，Datomic 团队在不断壮大和蓬勃发展。'
```


您还可以使用 OPML 文件，例如 Feedly 导出。传入 URL 或 OPML 内容。


```python
with open("example_data/sample_rss_feeds.opml", "r") as f:
    loader = RSSFeedLoader(opml=f.read())
data = loader.load()
print(len(data))
```
```output
Error fetching http://www.engadget.com/rss-full.xml, exception: Error fetching http://www.engadget.com/rss-full.xml, exception: document declared as us-ascii, but parsed as utf-8
``````output
20
```

```python
data[0].page_content
```



```output
'电动汽车初创公司 Fisker 昨晚在亨廷顿海滩引起了轰动，展示了一系列计划与 Fisker Ocean 一起生产的新电动车，该车正在缓慢开始在欧洲和美国交付。似乎有适合大多数人品味的车型，包括一款强大的四门 GT、一款多功能皮卡和一款实惠的电动城市车。\n\n“我们希望世界知道我们有宏伟的计划，并打算进入几个不同的细分市场，用我们独特的设计、创新和可持续性重新定义每个市场，”首席执行官 Henrik Fisker 说。\n\n首先是最便宜的 Fisker PEAR——一个“个人电动汽车革命”的可爱缩写——据说使用的零件比其他小型电动车少 35%。虽然它是一款小车，但 PEAR 拥有前后长椅座位，可以容纳六人。哦，它还有一个前备厢，公司称之为“froot”，这会让一些说英语的英国人满意，比如 Ars 的朋友和汽车记者 Jonny Smith。\n\n但最令人兴奋的是价格——起价 29,900 美元，计划于 2025 年上市。Fisker 计划与富士康合作在俄亥俄州的洛德斯敦生产 PEAR，这意味着它将符合联邦税收优惠的条件。\n\n广告\n\nFisker Alaska 是公司的皮卡，基于 Ocean 使用的改进版平台建造。它有一个可伸缩的货物床，长度可以是 4.5 英尺（1,371 毫米）或多达 9.2 英尺（2,804 毫米）。Fisker 声称它将是市场上最轻的电动皮卡，也是世界上最可持续的皮卡。续航里程预计为 230–240 英里（370–386 公里）。\n\n这款车也定于 2025 年上市，价格相对实惠，起价为 45,400 美元。Fisker 希望在北美生产这款车，但尚未透露具体地点。\n\n最后是 Ronin，这是一款四门 GT，与 Fisker Karma（Henrik Fisker 于 2012 年创造的车型）有着显著的相似之处。这款车的价格尚未公布，但 Fisker 表示其全轮驱动动力系统将拥有 1,000 马力（745 千瓦），并将在两秒内从静止加速到 60 英里每小时——几乎快到现代轮胎所能承受的极限。预计这款车将配备大容量电池，Fisker 表示其目标是达到 600 英里（956 公里）的续航里程。\n\n“创新和可持续性以及设计是我们的三大品牌价值。到 2027 年，我们打算生产世界上第一款气候中立的车辆，随着我们的客户重新定义与出行的关系，我们希望成为软件定义交通的领导者，”Fisker 说。'
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)