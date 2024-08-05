---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/kay.ipynb
---

# Kay.ai

>[Kai Data API](https://www.kay.ai/) 为 RAG 🕵️ 构建。我们正在策划世界上最大的高质量嵌入数据集，以便您的 AI 代理能够实时检索上下文。最新模型，快速检索，零基础设施。

本笔记本向您展示如何检索 [Kay](https://kay.ai/) 支持的数据集。目前，您可以搜索 `SEC Filings` 和 `Press Releases of US companies`。访问 [kay.ai](https://kay.ai) 获取最新数据发布。如有任何问题，请加入我们的 [discord](https://discord.gg/hAnE4e5T6M) 或 [tweet at us](https://twitter.com/vishalrohra_)

## 安装

首先，安装 [`kay` 包](https://pypi.org/project/kay/)。

```python
!pip install kay
```

您还需要一个 API 密钥：您可以在 [https://kay.ai](https://kay.ai/) 免费获取一个。获得 API 密钥后，您必须将其设置为环境变量 `KAY_API_KEY`。

`KayAiRetriever` 有一个静态 `.create()` 工厂方法，接受以下参数：

* `dataset_id: string` 必需 -- 一个 Kay 数据集 ID。这是关于特定实体（如公司、个人或地点）的数据集合。例如，尝试 `"company"` 
* `data_type: List[string]` 可选 -- 这是基于其来源或格式的数据集中的一个类别，例如“SEC 文件”、“新闻稿”或“报告”在“公司”数据集中。例如，尝试在“公司”数据集下使用 ["10-K", "10-Q", "PressRelease"]。如果留空，Kay 将检索所有类型中最相关的上下文。
* `num_contexts: int` 可选，默认为 6 -- 每次调用 `get_relevant_documents()` 时要检索的文档块数量。

## 示例

### 基本检索器使用

```python
# 设置API密钥
from getpass import getpass

KAY_API_KEY = getpass()
```
```output
 ········
```

```python
import os

from langchain_community.retrievers import KayAiRetriever

os.environ["KAY_API_KEY"] = KAY_API_KEY
retriever = KayAiRetriever.create(
    dataset_id="company", data_types=["10-K", "10-Q", "PressRelease"], num_contexts=3
)
docs = retriever.invoke(
    "2023年Roku在战略变更和合作伙伴关系方面最大的变化是什么？"
)
```

```python
docs
```

```output
[Document(page_content='公司名称：ROKU INC\n公司行业：有线电视及其他付费电视服务\n文章标题：Roku被评为《快速公司》2023年最具创新公司之一\n文本：该公司推出了几款新设备，包括Roku Voice Remote Pro；升级了其最顶级的播放器Roku Ultra；并推出了一系列新的智能家居设备，如视频门铃、灯具和集成到Roku生态系统中的插头。最近，该公司宣布将在今年春季推出Roku品牌电视，为消费者和Roku电视合作伙伴提供更多选择和创新。在2022年，Roku还更新了其操作系统（OS），这是唯一为电视专门设计的操作系统，增加了更多个性化功能，并在搜索、音频和内容发现方面进行了增强，推出了The Buzz、Sports和What to Watch，提供量身定制的电影和电视推荐，显示在主屏幕菜单上。该公司还为流媒体用户推出了一项新功能Photo Streams，允许客户通过Roku流媒体设备展示和分享相册。此外，Roku推出了可购物广告，这是一项新的广告创新，使在电视流媒体上购物变得像在社交媒体上一样简单。观众只需按下Roku遥控器上的“OK”键即可在可购物广告上进行购物，并通过Roku Pay这一专有支付平台预填的运输和支付信息完成结账。沃尔玛是此次推出的独家零售商，这是首个此类合作关系。', metadata={'chunk_type': 'text', 'chunk_years_mentioned': [2022, 2023], 'company_name': 'ROKU INC', 'company_sic_code_description': '有线电视及其他付费电视服务', 'data_source': 'PressRelease', 'data_source_link': 'https://newsroom.roku.com/press-releases', 'data_source_publish_date': '2023-03-02T09:30:00-04:00', 'data_source_uid': '963d4a81-f58e-3093-af68-987fb1758c15', 'title': "ROKU INC |  Roku被评为《快速公司》2023年最具创新公司之一"}),
 Document(page_content='公司名称：ROKU INC\n公司行业：有线电视及其他付费电视服务\n文章标题：Roku被评为《快速公司》2023年最具创新公司之一\n文本：最后，Roku扩展了其内容提供，用户可以选择数千个应用程序和观看选项，包括Roku频道上的内容，该应用在2022年成为美国Roku平台上前五名的应用之一。11月，Roku发布了其首部故事片《怪异：怪异阿尔·扬科维奇的故事》，主演为丹尼尔·拉德克里夫。在这一年里，Roku频道增加了来自NBC环球和国家冰球联盟的快速频道，以及一个独家的AMC频道，播放其标志性剧集《广告狂人》。今年，该公司宣布与华纳兄弟探索达成协议，推出的新频道将包括《西部世界》和《单身汉》，以及2000小时的点播内容。有关Roku的发展历程，请点击此处查看。《快速公司》最具创新公司特刊（2023年3/4月）可在线查看，亦可通过iTunes应用内查看，并于3月14日起在报摊上发售。关于Roku, Inc.\nRoku开创了电视流媒体的先河。我们将用户与他们喜爱的流媒体内容连接起来，使内容发布者能够建立和盈利于大量受众，并为广告商提供独特的能力以吸引消费者。Roku流媒体播放器和电视相关音频设备在美国及部分国家通过直接零售销售和与服务运营商的许可协议提供。Roku电视型号在美国及部分国家通过与电视OEM品牌的许可协议提供。', metadata={'chunk_type': 'text', 'chunk_years_mentioned': [2022, 2023], 'company_name': 'ROKU INC', 'company_sic_code_description': '有线电视及其他付费电视服务', 'data_source': 'PressRelease', 'data_source_link': 'https://newsroom.roku.com/press-releases', 'data_source_publish_date': '2023-03-02T09:30:00-04:00', 'data_source_uid': '963d4a81-f58e-3093-af68-987fb1758c15', 'title': "ROKU INC |  Roku被评为《快速公司》2023年最具创新公司之一"}),
 Document(page_content='公司名称：ROKU INC\n公司行业：有线电视及其他付费电视服务\n文章标题：Roku的新NFL专区让粉丝轻松访问NFL比赛，正好赶上2023赛季\n文本：与NFL合作，新NFL专区为观众提供了一个轻松找到观看NFL直播比赛的方式。今天，Roku（纳斯达克：ROKU）和国家橄榄球联盟（NFL）宣布，在Roku体育体验中推出的NFL专区，以开启2023 NFL赛季。Roku和NFL之间的这一战略合作标志着Roku体育体验中首次官方联赛品牌专区的推出。NFL专区现已上线，为足球迷提供了一个集中位置，以查找直播和即将到来的比赛，让他们可以减少寻找观看比赛的时间，更多地关注为自己喜欢的球队加油。用户还可以收看每周比赛预览、联赛亮点以及其他NFL内容，所有内容均在专区内。此新闻稿包含多媒体。请在此处查看完整发布内容：与NFL合作，Roku的新NFL专区为观众提供了一个轻松找到观看NFL直播比赛的方式（照片：商业电讯）。“去年，我们为高度参与的体育观众推出了体育体验，使Roku用户更容易观看体育节目，”Roku消费者体验部总裁Gidon Katz表示。“随着我们开始一年中最大的体育赛季，为我们的数百万用户提供轻松访问NFL比赛和内容是我们的首要任务。我们期待粉丝们沉浸在NFL专区，并将其作为寻找NFL比赛的目的地。', metadata={'chunk_type': 'text', 'chunk_years_mentioned': [2023], 'company_name': 'ROKU INC', 'company_sic_code_description': '有线电视及其他付费电视服务', 'data_source': 'PressRelease', 'data_source_link': 'https://newsroom.roku.com/press-releases', 'data_source_publish_date': '2023-09-12T09:00:00-04:00', 'data_source_uid': '963d4a81-f58e-3093-af68-987fb1758c15', 'title': "ROKU INC |  Roku的新NFL专区让粉丝轻松访问NFL比赛，正好赶上2023赛季"})]
```

### 链中的用法


```python
OPENAI_API_KEY = getpass()
```
```output
 ········
```

```python
os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```


```python
from langchain.chains import ConversationalRetrievalChain
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-3.5-turbo")
qa = ConversationalRetrievalChain.from_llm(model, retriever=retriever)
```


```python
questions = [
    "2023年Roku最大的战略变化和合作伙伴关系是什么？"
    # "2023年Wex在哪些方面赚取最多的利润？",
]
chat_history = []

for question in questions:
    result = qa({"question": question, "chat_history": chat_history})
    chat_history.append((question, result["answer"]))
    print(f"-> **问题**: {question} \n")
    print(f"**答案**: {result['answer']} \n")
```
```output
-> **问题**: 2023年Roku最大的战略变化和合作伙伴关系是什么？ 

**答案**: 在2023年，Roku与FreeWheel达成战略合作伙伴关系，将Roku领先的广告技术引入FreeWheel客户。这一合作旨在推动广告视频点播（AVOD）领域的更大互操作性和自动化。此合作的主要亮点包括Roku的需求应用程序编程接口（dAPI）与FreeWheel的电视平台的简化集成，从而实现更好的库存质量控制和提高出版商的收益。此外，出版商现在可以使用Roku平台信号，使广告主能够针对受众并衡量广告活动表现，而无需依赖cookies。该合作还涉及数据清理室技术的使用，以便激活额外的数据集，从而为出版商和代理商提供更好的测量和货币化。这些合作伙伴关系和战略旨在支持Roku在AVOD市场的增长。
```

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)