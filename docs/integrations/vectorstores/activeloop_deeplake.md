---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/activeloop_deeplake.ipynb
---

# Activeloop Deep Lake

>[Activeloop Deep Lake](https://docs.activeloop.ai/) 是一个多模态向量存储，能够存储嵌入及其元数据，包括文本、Json、图像、音频、视频等。它可以将数据保存在本地、云端或 Activeloop 存储中。它执行混合搜索，包括嵌入及其属性。

本笔记本展示了与 `Activeloop Deep Lake` 相关的基本功能。虽然 `Deep Lake` 可以存储嵌入，但它能够存储任何类型的数据。它是一个无服务器的数据湖，具有版本控制、查询引擎和流式数据加载器，适用于深度学习框架。

有关更多信息，请参见 Deep Lake [文档](https://docs.activeloop.ai) 或 [API 参考](https://docs.deeplake.ai)

## 设置

```python
%pip install --upgrade --quiet  langchain-openai langchain-community 'deeplake[enterprise]' tiktoken
```

## Activeloop 提供的示例

[与 LangChain 的集成](https://docs.activeloop.ai/tutorials/vector-store/deep-lake-vector-store-in-langchain).

## Deep Lake 本地

```python
from langchain_community.vectorstores import DeepLake
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
activeloop_token = getpass.getpass("activeloop token:")
embeddings = OpenAIEmbeddings()
```

```python
from langchain_community.document_loaders import TextLoader

loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()
```

### 创建本地数据集

在 `./deeplake/` 本地创建数据集，然后运行相似性搜索。Deeplake+LangChain 集成在底层使用 Deep Lake 数据集，因此 `dataset` 和 `vector store` 可以互换使用。要在您自己的云中或在 Deep Lake 存储中创建数据集，请 [相应地调整路径](https://docs.activeloop.ai/storage-and-credentials/storage-options)。

```python
db = DeepLake(dataset_path="./my_deeplake/", embedding=embeddings, overwrite=True)
db.add_documents(docs)
# or shorter
# db = DeepLake.from_documents(docs, dataset_path="./my_deeplake/", embedding=embeddings, overwrite=True)
```

### 查询数据集

```python
query = "总统关于凯坦吉·布朗·杰克逊说了什么"
docs = db.similarity_search(query)
```
```output

``````output
Dataset(path='./my_deeplake/', tensors=['embedding', 'id', 'metadata', 'text'])

  tensor      htype      shape      dtype  compression
  -------    -------    -------    -------  ------- 
 embedding  embedding  (42, 1536)  float32   None   
    id        text      (42, 1)      str     None   
 metadata     json      (42, 1)      str     None   
   text       text      (42, 1)      str     None
``````output

```
要禁用数据集摘要的持续打印，可以在初始化 VectorStore 时指定 verbose=False。

```python
print(docs[0].page_content)
```
```output
今晚。我呼吁参议院：通过《投票自由法》。通过《约翰·刘易斯投票权法》。在此期间，通过《披露法》，让美国人知道谁在资助我们的选举。

今晚，我想表彰一位为这个国家奉献一生的人：大法官斯蒂芬·布雷耶——一位退伍军人、宪法学者以及即将退休的美国最高法院大法官。布雷耶大法官，感谢您的服务。

总统最重要的宪法责任之一是提名某人担任美国最高法院法官。

而我在4天前做到了这一点，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律人才之一，将继续布雷耶大法官卓越的遗产。
```
之后，您可以在不重新计算嵌入的情况下重新加载数据集。

```python
db = DeepLake(dataset_path="./my_deeplake/", embedding=embeddings, read_only=True)
docs = db.similarity_search(query)
```
```output
Deep Lake 数据集在 ./my_deeplake/ 中已存在，从存储中加载
```
Deep Lake 目前是单写入和多读取。设置 `read_only=True` 有助于避免获取写入锁。

### 检索问答

```python
from langchain.chains import RetrievalQA
from langchain_openai import OpenAIChat

qa = RetrievalQA.from_chain_type(
    llm=OpenAIChat(model="gpt-3.5-turbo"),
    chain_type="stuff",
    retriever=db.as_retriever(),
)
```
```output
/home/ubuntu/langchain_activeloop/langchain/libs/langchain/langchain/llms/openai.py:786: UserWarning: You are trying to use a chat model. This way of initializing it is no longer supported. Instead, please use: `from langchain_openai import ChatOpenAI`
  warnings.warn(
```

```python
query = "总统对凯坦吉·布朗·杰克逊说了什么"
qa.run(query)
```

```output
'总统表示，凯坦吉·布朗·杰克逊是前顶级私人执业律师和前联邦公设辩护人。她来自一个公立学校教育工作者和警察的家庭。她是一位共识建设者，自被提名以来获得了广泛的支持。'
```

### 基于属性的元数据过滤

我们来创建另一个包含文档创建年份的向量存储。

```python
import random

for d in docs:
    d.metadata["year"] = random.randint(2012, 2014)

db = DeepLake.from_documents(
    docs, embeddings, dataset_path="./my_deeplake/", overwrite=True
)
```
```output

``````output
Dataset(path='./my_deeplake/', tensors=['embedding', 'id', 'metadata', 'text'])

  tensor      htype      shape     dtype  compression
  -------    -------    -------   -------  ------- 
 embedding  embedding  (4, 1536)  float32   None   
    id        text      (4, 1)      str     None   
 metadata     json      (4, 1)      str     None   
   text       text      (4, 1)      str     None
``````output

```

```python
db.similarity_search(
    "总统对凯坦吉·布朗·杰克逊说了什么",
    filter={"metadata": {"year": 2013}},
)
```
```output
100%|██████████| 4/4 [00:00<00:00, 2936.16it/s]
```


```output
[Document(page_content='今晚。我呼吁参议院：通过《投票自由法》。通过《约翰·刘易斯投票权法》。在此期间，通过《披露法》，让美国人知道谁在资助我们的选举。\n\n今晚，我想表彰一位为这个国家奉献一生的人：大法官斯蒂芬·布雷耶——一位退伍军人、宪法学者，以及即将退休的美国最高法院大法官。布雷耶法官，感谢您的服务。\n\n总统最严肃的宪法责任之一就是提名某人担任美国最高法院大法官。\n\n而我在4天前做了这件事，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律人才之一，将继续布雷耶法官的卓越遗产。', metadata={'source': '../../how_to/state_of_the_union.txt', 'year': 2013}),
 Document(page_content='一位曾在私人执业中担任顶级诉讼律师的前联邦公设辩护人。来自公立学校教育工作者和警察家庭的共识建立者。自从她被提名以来，她获得了广泛的支持——从警察兄弟会到由民主党和共和党任命的前法官。\n\n如果我们要推进自由与正义，我们需要确保边界安全并修复移民系统。\n\n我们可以做到这两点。在我们的边界，我们安装了新的技术，比如尖端扫描仪，以更好地检测毒品走私。\n\n我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人贩子。\n\n我们正在设立专门的移民法官，以便逃离迫害和暴力的家庭能够更快地听取他们的案件。\n\n我们正在确保承诺并支持南美和中美洲的合作伙伴，接纳更多难民并确保他们自己的边界安全。', metadata={'source': '../../how_to/state_of_the_union.txt', 'year': 2013}),
 Document(page_content='今晚，我宣布对这些向美国企业和消费者收取过高费用的公司进行打击。\n\n随着华尔街公司接管更多的养老院，这些养老院的质量下降，费用上升。\n\n在我任内，这种情况将结束。\n\n医疗保险将为养老院设定更高的标准，确保您的亲人获得他们应得的护理。\n\n我们还将通过给予工人公平的机会来降低成本，提供更多培训和学徒机会，根据技能而不是学位来雇佣他们。\n\n让我们通过《工资公平法》和带薪休假。\n\n将最低工资提高到每小时15美元，并延长儿童税收抵免，以便没有人需要在贫困中抚养家庭。\n\n让我们增加佩尔助学金，并增加我们对历史上黑人大学的支持，投资于我们第一夫人吉尔称之为美国最佳秘密的社区大学。', metadata={'source': '../../how_to/state_of_the_union.txt', 'year': 2013})]
```

### 选择距离函数
距离函数 `L2` 表示欧几里得距离，`L1` 表示核范数，`Max` 表示无穷大距离，`cos` 表示余弦相似度，`dot` 表示点积

```python
db.similarity_search(
    "What did the president say about Ketanji Brown Jackson?", distance_metric="cos"
)
```

```output
[Document(page_content='Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. \n\nTonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={'source': '../../how_to/state_of_the_union.txt', 'year': 2013}),
 Document(page_content='A former top litigator in private practice. A former federal public defender. And from a family of public school educators and police officers. A consensus builder. Since she’s been nominated, she’s received a broad range of support—from the Fraternal Order of Police to former judges appointed by Democrats and Republicans. \n\nAnd if we are to advance liberty and justice, we need to secure the Border and fix the immigration system. \n\nWe can do both. At our border, we’ve installed new technology like cutting-edge scanners to better detect drug smuggling.  \n\nWe’ve set up joint patrols with Mexico and Guatemala to catch more human traffickers.  \n\nWe’re putting in place dedicated immigration judges so families fleeing persecution and violence can have their cases heard faster. \n\nWe’re securing commitments and supporting partners in South and Central America to host more refugees and secure their own borders.', metadata={'source': '../../how_to/state_of_the_union.txt', 'year': 2013}),
 Document(page_content='Tonight, I’m announcing a crackdown on these companies overcharging American businesses and consumers. \n\nAnd as Wall Street firms take over more nursing homes, quality in those homes has gone down and costs have gone up.  \n\nThat ends on my watch. \n\nMedicare is going to set higher standards for nursing homes and make sure your loved ones get the care they deserve and expect. \n\nWe’ll also cut costs and keep the economy going strong by giving workers a fair shot, provide more training and apprenticeships, hire them based on their skills not degrees. \n\nLet’s pass the Paycheck Fairness Act and paid leave.  \n\nRaise the minimum wage to $15 an hour and extend the Child Tax Credit, so no one has to raise a family in poverty. \n\nLet’s increase Pell Grants and increase our historic support of HBCUs, and invest in what Jill—our First Lady who teaches full-time—calls America’s best-kept secret: community colleges.', metadata={'source': '../../how_to/state_of_the_union.txt', 'year': 2013}),
 Document(page_content='And for our LGBTQ+ Americans, let’s finally get the bipartisan Equality Act to my desk. The onslaught of state laws targeting transgender Americans and their families is wrong. \n\nAs I said last year, especially to our younger transgender Americans, I will always have your back as your President, so you can be yourself and reach your God-given potential. \n\nWhile it often appears that we never agree, that isn’t true. I signed 80 bipartisan bills into law last year. From preventing government shutdowns to protecting Asian-Americans from still-too-common hate crimes to reforming military justice. \n\nAnd soon, we’ll strengthen the Violence Against Women Act that I first wrote three decades ago. It is important for us to show the nation that we can come together and do big things. \n\nSo tonight I’m offering a Unity Agenda for the Nation. Four big things we can do together.  \n\nFirst, beat the opioid epidemic.', metadata={'source': '../../how_to/state_of_the_union.txt', 'year': 2012})]
```

### 最大边际相关性
使用最大边际相关性


```python
db.max_marginal_relevance_search(
    "总统对 Ketanji Brown Jackson 说了什么？"
)
```



```output
[Document(page_content='今晚。我呼吁参议院：通过《投票自由法》。通过《约翰·刘易斯投票权法》。同时，请通过《披露法》，让美国人知道是谁在资助我们的选举。\n\n今晚，我想表彰一位为这个国家奉献一生的人：大法官斯蒂芬·布雷耶——一位退伍军人，宪法学者，以及即将退休的美国最高法院大法官。布雷耶大法官，感谢您的服务。\n\n总统最严肃的宪法责任之一就是提名某人担任美国最高法院法官。\n\n我在四天前做到了这一点，当时我提名了上诉法院法官 Ketanji Brown Jackson。她是我们国家顶尖的法律人才之一，将延续布雷耶大法官的卓越遗产。', metadata={'source': '../../how_to/state_of_the_union.txt', 'year': 2013}),
 Document(page_content='今晚，我宣布对这些向美国企业和消费者收取过高费用的公司进行打击。\n\n随着华尔街公司接管越来越多的养老院，这些养老院的质量下降，费用却上升。\n\n这一切在我任内将结束。\n\n医疗保险将设定更高的养老院标准，确保您的亲人获得应有的照顾。\n\n我们还将降低成本，保持经济强劲，通过给予工人公平的机会，提供更多的培训和学徒机会，根据他们的技能而非学位来雇佣他们。\n\n让我们通过《薪资公平法》和带薪休假。\n\n将最低工资提高到每小时 15 美元，延长儿童税收抵免，让没有人需要在贫困中抚养家庭。\n\n让我们增加佩尔助学金，并增加我们对历史黑人学院和大学的支持，投资于 Jill——我们全职任教的第一夫人——所称的美国最佳秘密：社区大学。', metadata={'source': '../../how_to/state_of_the_union.txt', 'year': 2013}),
 Document(page_content='一位曾在私人执业中担任顶级诉讼律师的前任。前联邦公设辩护人。来自一户公立学校教育工作者和警察的家庭。一个共识建设者。自从她被提名以来，她得到了广泛的支持——从警察兄弟会到民主党和共和党任命的前法官。\n\n如果我们要促进自由和正义，我们需要保卫边界并修复移民系统。\n\n我们可以同时做到这两点。在我们的边界，我们安装了新技术，如尖端扫描仪，以更好地检测毒品走私。\n\n我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人口贩子。\n\n我们正在设立专门的移民法官，以便逃离迫害和暴力的家庭能够更快地审理他们的案件。\n\n我们正在确保承诺并支持南美和中美洲的合作伙伴，接纳更多难民并保卫他们自己的边界。', metadata={'source': '../../how_to/state_of_the_union.txt', 'year': 2013}),
 Document(page_content='对于我们的 LGBTQ+ 美国人，让我们最终将两党支持的平等法案送到我的桌子上。针对跨性别美国人及其家庭的州法律的攻击是错误的。\n\n正如我去年所说，特别是对我们年轻的跨性别美国人，我作为你们的总统将始终支持你们，让你们能够做自己，实现上帝赋予你们的潜力。\n\n虽然看起来我们从未达成一致，但事实并非如此。我去年签署了 80 项两党法案。从防止政府停摆到保护亚裔美国人免受仍然过于普遍的仇恨犯罪，再到改革军事司法。\n\n不久之后，我们将加强我三十年前首次撰写的《反对暴力法》。我们必须向全国展示，我们可以团结在一起，做出重大成就。\n\n所以今晚我提出一个国家团结议程。我们可以共同做的四件大事。\n\n首先，打击阿片类药物流行病。', metadata={'source': '../../how_to/state_of_the_union.txt', 'year': 2012})]
```

### 删除数据集


```python
db.delete_dataset()
```
```output

```
如果删除失败，您也可以强制删除


```python
DeepLake.force_delete_by_path("./my_deeplake")
```
```output

```

## Deep Lake 数据集在云端（Activeloop、AWS、GCS 等）或内存中
默认情况下，Deep Lake 数据集存储在本地。要将其存储在内存中、Deep Lake 管理的数据库中或任何对象存储中，您可以在创建向量存储时提供[相应的路径和凭据](https://docs.activeloop.ai/storage-and-credentials/storage-options)。某些路径需要在 Activeloop 注册并创建一个 API 令牌，该令牌可以在[此处检索](https://app.activeloop.ai/)


```python
os.environ["ACTIVELOOP_TOKEN"] = activeloop_token
```


```python
# 嵌入并存储文本
username = "<USERNAME_OR_ORG>"  # 您在 app.activeloop.ai 上的用户名
dataset_path = f"hub://{username}/langchain_testing_python"  # 也可以是 ./local/path（本地速度更快），s3://bucket/path/to/dataset，gcs://path/to/dataset 等。

docs = text_splitter.split_documents(documents)

embedding = OpenAIEmbeddings()
db = DeepLake(dataset_path=dataset_path, embedding=embeddings, overwrite=True)
ids = db.add_documents(docs)
```
```output
您的 Deep Lake 数据集已成功创建！
``````output

``````output
Dataset(path='hub://adilkhan/langchain_testing_python', tensors=['embedding', 'id', 'metadata', 'text'])

  tensor      htype      shape      dtype  compression
  -------    -------    -------    -------  ------- 
 embedding  embedding  (42, 1536)  float32   None   
    id        text      (42, 1)      str     None   
 metadata     json      (42, 1)      str     None   
   text       text      (42, 1)      str     None
``````output

```

```python
query = "总统对凯坦吉·布朗·杰克逊说了什么"
docs = db.similarity_search(query)
print(docs[0].page_content)
```
```output
今晚。我呼吁参议院：通过《投票自由法案》。通过《约翰·刘易斯投票权法案》。而在您做这些的时候，通过《披露法案》，让美国人知道谁在资助我们的选举。

今晚，我想表彰一位为这个国家奉献一生的人：斯蒂芬·布雷耶大法官——一位退伍军人、宪法学者，以及即将退休的美国最高法院大法官。布雷耶大法官，感谢您的服务。

总统最重要的宪法责任之一是提名某人担任美国最高法院大法官。

我在四天前做到了这一点，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律头脑之一，将继续布雷耶大法官卓越的遗产。
```
#### `tensor_db` 执行选项 

为了利用 Deep Lake 的管理张量数据库，在创建向量存储时需要将运行时参数指定为 {'tensor_db': True}。此配置允许在管理张量数据库上执行查询，而不是在客户端执行。需要注意的是，此功能不适用于存储在本地或内存中的数据集。如果向量存储已经在管理张量数据库之外创建，可以按照规定的步骤将其转移到管理张量数据库中。


```python
# 嵌入并存储文本
username = "<USERNAME_OR_ORG>"  # 您在 app.activeloop.ai 上的用户名
dataset_path = f"hub://{username}/langchain_testing"

docs = text_splitter.split_documents(documents)

embedding = OpenAIEmbeddings()
db = DeepLake(
    dataset_path=dataset_path,
    embedding=embeddings,
    overwrite=True,
    runtime={"tensor_db": True},
)
ids = db.add_documents(docs)
```
```output
您的 Deep Lake 数据集已成功创建！
``````output
|
``````output
Dataset(path='hub://adilkhan/langchain_testing', tensors=['embedding', 'id', 'metadata', 'text'])

  tensor      htype      shape      dtype  compression
  -------    -------    -------    -------  ------- 
 embedding  embedding  (42, 1536)  float32   None   
    id        text      (42, 1)      str     None   
 metadata     json      (42, 1)      str     None   
   text       text      (42, 1)      str     None
``````output

```

### TQL搜索

此外，查询的执行也支持在similarity_search方法中，可以利用Deep Lake的张量查询语言（TQL）指定查询。

```python
search_id = db.vectorstore.dataset.id[0].numpy()
```

```python
search_id[0]
```

```output
'8a6ff326-3a85-11ee-b840-13905694aaaf'
```

```python
docs = db.similarity_search(
    query=None,
    tql=f"SELECT * WHERE id == '{search_id[0]}'",
)
```

```python
db.vectorstore.summary()
```
```output
Dataset(path='hub://adilkhan/langchain_testing', tensors=['embedding', 'id', 'metadata', 'text'])

  tensor      htype      shape      dtype  compression
  -------    -------    -------    -------  ------- 
 embedding  embedding  (42, 1536)  float32   None   
    id        text      (42, 1)      str     None   
 metadata     json      (42, 1)      str     None   
   text       text      (42, 1)      str     None
```

### 在 AWS S3 上创建向量存储


```python
dataset_path = "s3://BUCKET/langchain_test"  # 也可以是 ./local/path（本地速度更快），hub://bucket/path/to/dataset，gcs://path/to/dataset 等。

embedding = OpenAIEmbeddings()
db = DeepLake.from_documents(
    docs,
    dataset_path=dataset_path,
    embedding=embeddings,
    overwrite=True,
    creds={
        "aws_access_key_id": os.environ["AWS_ACCESS_KEY_ID"],
        "aws_secret_access_key": os.environ["AWS_SECRET_ACCESS_KEY"],
        "aws_session_token": os.environ["AWS_SESSION_TOKEN"],  # 可选
    },
)
```
```output
s3://hub-2.0-datasets-n/langchain_test 加载成功。
``````output
评估摄取：100%|██████████| 1/1 [00:10<00:00
\
``````output
数据集(path='s3://hub-2.0-datasets-n/langchain_test', tensors=['embedding', 'ids', 'metadata', 'text'])

  tensor     htype     shape     dtype  compression
  -------   -------   -------   -------  ------- 
 embedding  generic  (4, 1536)  float32   None   
    ids      text     (4, 1)      str     None   
 metadata    json     (4, 1)      str     None   
   text      text     (4, 1)      str     None
``````output

```

## Deep Lake API
您可以通过 `db.vectorstore` 访问 Deep Lake 数据集


```python
# get structure of the dataset
db.vectorstore.summary()
```
```output
Dataset(path='hub://adilkhan/langchain_testing', tensors=['embedding', 'id', 'metadata', 'text'])

  tensor      htype      shape      dtype  compression
  -------    -------    -------    -------  ------- 
 embedding  embedding  (42, 1536)  float32   None   
    id        text      (42, 1)      str     None   
 metadata     json      (42, 1)      str     None   
   text       text      (42, 1)      str     None
```

```python
# get embeddings numpy array
embeds = db.vectorstore.dataset.embedding.numpy()
```

### 将本地数据集转移到云端
将已创建的数据集复制到云端。您也可以从云端转移到本地。

```python
import deeplake

username = "davitbun"  # your username on app.activeloop.ai
source = f"hub://{username}/langchain_testing"  # could be local, s3, gcs, etc.
destination = f"hub://{username}/langchain_test_copy"  # could be local, s3, gcs, etc.

deeplake.deepcopy(src=source, dest=destination, overwrite=True)
```
```output
Copying dataset: 100%|██████████| 56/56 [00:38<00:00
``````output
此数据集可以通过 ds.visualize() 在 Jupyter Notebook 中可视化，或访问 https://app.activeloop.ai/davitbun/langchain_test_copy
您的 Deep Lake 数据集已成功创建！
该数据集是私有的，请确保您已登录！
```


```output
Dataset(path='hub://davitbun/langchain_test_copy', tensors=['embedding', 'ids', 'metadata', 'text'])
```



```python
db = DeepLake(dataset_path=destination, embedding=embeddings)
db.add_documents(docs)
```
```output

``````output
此数据集可以通过 ds.visualize() 在 Jupyter Notebook 中可视化，或访问 https://app.activeloop.ai/davitbun/langchain_test_copy
``````output
/
``````output
hub://davitbun/langchain_test_copy 加载成功。
``````output
Deep Lake 数据集在 hub://davitbun/langchain_test_copy 已存在，正在从存储中加载
``````output
Dataset(path='hub://davitbun/langchain_test_copy', tensors=['embedding', 'ids', 'metadata', 'text'])

  tensor     htype     shape     dtype  compression
  -------   -------   -------   -------  ------- 
 embedding  generic  (4, 1536)  float32   None   
    ids      text     (4, 1)      str     None   
 metadata    json     (4, 1)      str     None   
   text      text     (4, 1)      str     None
``````output
评估摄取： 100%|██████████| 1/1 [00:31<00:00
-
``````output
Dataset(path='hub://davitbun/langchain_test_copy', tensors=['embedding', 'ids', 'metadata', 'text'])

  tensor     htype     shape     dtype  compression
  -------   -------   -------   -------  ------- 
 embedding  generic  (8, 1536)  float32   None   
    ids      text     (8, 1)      str     None   
 metadata    json     (8, 1)      str     None   
   text      text     (8, 1)      str     None
``````output

```


```output
['ad42f3fe-e188-11ed-b66d-41c5f7b85421',
 'ad42f3ff-e188-11ed-b66d-41c5f7b85421',
 'ad42f400-e188-11ed-b66d-41c5f7b85421',
 'ad42f401-e188-11ed-b66d-41c5f7b85421']
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)