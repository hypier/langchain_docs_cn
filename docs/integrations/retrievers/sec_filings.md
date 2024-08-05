---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/sec_filings.ipynb
---

# SEC备案


>[SEC备案](https://www.sec.gov/edgar) 是提交给美国证券交易委员会（SEC）的财务报表或其他正式文件。上市公司、某些内部人士和经纪交易商需要定期进行 `SEC备案`。投资者和金融专业人士依赖这些备案获取他们评估投资目的的公司的信息。
>
>`SEC备案` 数据由 [Kay.ai](https://kay.ai) 和 [Cybersyn](https://www.cybersyn.com/) 通过 [Snowflake Marketplace](https://app.snowflake.com/marketplace/providers/GZTSZAS2KCS/Cybersyn%2C%20Inc) 提供。

## 设置

首先，您需要安装 `kay` 包。您还需要一个 API 密钥：您可以在 [https://kay.ai](https://kay.ai/) 免费获取一个。一旦您拥有 API 密钥，就必须将其设置为环境变量 `KAY_API_KEY`。

在这个例子中，我们将使用 `KayAiRetriever`。请查看 [kay notebook](/docs/integrations/retrievers/kay) 以获取有关它接受的参数的详细信息。

```python
# Setup API keys for Kay and OpenAI
from getpass import getpass

KAY_API_KEY = getpass()
OPENAI_API_KEY = getpass()
```
```output
 ········
 ········
```

```python
import os

os.environ["KAY_API_KEY"] = KAY_API_KEY
os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```

## 示例


```python
from langchain.chains import ConversationalRetrievalChain
from langchain_community.retrievers import KayAiRetriever
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-3.5-turbo")
retriever = KayAiRetriever.create(
    dataset_id="company", data_types=["10-K", "10-Q"], num_contexts=6
)
qa = ConversationalRetrievalChain.from_llm(model, retriever=retriever)
```


```python
questions = [
    "Nvidia在过去三个季度的支出模式是什么？",
    # "可再生能源行业最近面临的挑战是什么？",
]
chat_history = []

for question in questions:
    result = qa({"question": question, "chat_history": chat_history})
    chat_history.append((question, result["answer"]))
    print(f"-> **问题**: {question} \n")
    print(f"**回答**: {result['answer']} \n")
```
```output
-> **问题**: Nvidia在过去三个季度的支出模式是什么？ 

**回答**: 根据提供的信息，以下是NVIDIA在过去三个季度的支出模式：

1. 研究与开发费用：
   - 2022年第三季度：与2021年第三季度相比增加了34%。
   - 2023年第一季度：与2022年第一季度相比增加了40%。
   - 2022年第二季度：与2021年第二季度相比增加了25%。
   
   总体而言，研究与开发费用在过去三个季度中持续增加。

2. 销售、一般和管理费用：
   - 2022年第三季度：与2021年第三季度相比增加了8%。
   - 2023年第一季度：与2022年第一季度相比增加了14%。
   - 2022年第二季度：与2021年第二季度相比减少了16%。
   
   销售、一般和管理费用的模式不那么一致，有些季度显示增加，有些季度显示减少。

3. 总运营费用：
   - 2022年第三季度：与2021年第三季度相比增加了25%。
   - 2023年第一季度：与2022年第一季度相比增加了113%。
   - 2022年第二季度：与2021年第二季度相比增加了9%。
   
   总运营费用在过去三个季度中普遍增加，2023年第一季度的增幅显著。

总体而言，模式表明研究与开发费用和总运营费用持续增加，而销售、一般和管理费用则显示出一些波动。
```

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)