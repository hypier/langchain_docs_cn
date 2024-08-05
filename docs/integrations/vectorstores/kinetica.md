---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/kinetica.ipynb
sidebar_label: Kinetica
---

# Kinetica Vectorstore API

>[Kinetica](https://www.kinetica.com/) 是一个具有集成向量相似性搜索支持的数据库

它支持：
- 精确和近似最近邻搜索
- L2 距离、内积和余弦距离

此笔记本展示了如何使用 Kinetica 向量存储（`Kinetica`）。

这需要一个 Kinetica 实例，可以通过此处提供的说明轻松设置 - [安装说明](https://www.kinetica.com/developer-edition/)。


```python
# Pip install necessary package
%pip install --upgrade --quiet  langchain-openai langchain-community
%pip install gpudb==7.2.0.9
%pip install --upgrade --quiet  tiktoken
```
```output

[1m[[0m[34;49mnotice[0m[1;39;49m][0m[39;49m A new release of pip is available: [0m[31;49m23.2.1[0m[39;49m -> [0m[32;49m24.0[0m
[1m[[0m[34;49mnotice[0m[1;39;49m][0m[39;49m To update, run: [0m[32;49mpip install --upgrade pip[0m
Note: you may need to restart the kernel to use updated packages.
Requirement already satisfied: gpudb==7.2.0.0b in /home/anindyam/kinetica/kinetica-github/langchain/libs/langchain/.venv/lib/python3.8/site-packages (7.2.0.0b0)
Requirement already satisfied: future in /home/anindyam/kinetica/kinetica-github/langchain/libs/langchain/.venv/lib/python3.8/site-packages (from gpudb==7.2.0.0b) (0.18.3)
Requirement already satisfied: pyzmq in /home/anindyam/kinetica/kinetica-github/langchain/libs/langchain/.venv/lib/python3.8/site-packages (from gpudb==7.2.0.0b) (25.1.2)

[1m[[0m[34;49mnotice[0m[1;39;49m][0m[39;49m A new release of pip is available: [0m[31;49m23.2.1[0m[39;49m -> [0m[32;49m24.0[0m
[1m[[0m[34;49mnotice[0m[1;39;49m][0m[39;49m To update, run: [0m[32;49mpip install --upgrade pip[0m
Note: you may need to restart the kernel to use updated packages.

[1m[[0m[34;49mnotice[0m[1;39;49m][0m[39;49m A new release of pip is available: [0m[31;49m23.2.1[0m[39;49m -> [0m[32;49m24.0[0m
[1m[[0m[34;49mnotice[0m[1;39;49m][0m[39;49m To update, run: [0m[32;49mpip install --upgrade pip[0m
Note: you may need to restart the kernel to use updated packages.
```
我们想使用 `OpenAIEmbeddings`，所以我们需要获取 OpenAI API 密钥。


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```


```python
## Loading Environment Variables
from dotenv import load_dotenv

load_dotenv()
```



```output
False
```



```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import (
    DistanceStrategy,
    Kinetica,
    KineticaSettings,
)
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```


```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()
```


```python
# Kinetica needs the connection to the database.
# This is how to set it up.
HOST = os.getenv("KINETICA_HOST", "http://127.0.0.1:9191")
USERNAME = os.getenv("KINETICA_USERNAME", "")
PASSWORD = os.getenv("KINETICA_PASSWORD", "")
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY", "")


def create_config() -> KineticaSettings:
    return KineticaSettings(host=HOST, username=USERNAME, password=PASSWORD)
```

## 使用欧几里得距离进行相似性搜索（默认）

```python
# Kinetica模块将尝试创建一个与集合名称相同的表。
# 因此，请确保集合名称是唯一的，并且用户有权限创建表。

COLLECTION_NAME = "state_of_the_union_test"
connection = create_config()

db = Kinetica.from_documents(
    embedding=embeddings,
    documents=docs,
    collection_name=COLLECTION_NAME,
    config=connection,
)
```

```python
query = "总统对Ketanji Brown Jackson说了什么"
docs_with_score = db.similarity_search_with_score(query)
```

```python
for doc, score in docs_with_score:
    print("-" * 80)
    print("Score: ", score)
    print(doc.page_content)
    print("-" * 80)
```
```output
--------------------------------------------------------------------------------
Score:  0.6077010035514832
今晚。我呼吁参议院：通过《投票自由法》。通过《约翰·刘易斯投票权法》。同时，请通过《披露法》，让美国人知道谁在资助我们的选举。

今晚，我想表彰一位为这个国家奉献一生的人：大法官斯蒂芬·布雷耶——一位退役军人，宪法学者，以及即将退休的美国最高法院大法官。布雷耶大法官，感谢您的服务。

总统最重要的宪法责任之一是提名某人担任美国最高法院法官。

我在4天前做了这件事，当时我提名了巡回上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律人才之一，将继续布雷耶大法官卓越的遗产。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.6077010035514832
今晚。我呼吁参议院：通过《投票自由法》。通过《约翰·刘易斯投票权法》。同时，请通过《披露法》，让美国人知道谁在资助我们的选举。

今晚，我想表彰一位为这个国家奉献一生的人：大法官斯蒂芬·布雷耶——一位退役军人，宪法学者，以及即将退休的美国最高法院大法官。布雷耶大法官，感谢您的服务。

总统最重要的宪法责任之一是提名某人担任美国最高法院法官。

我在4天前做了这件事，当时我提名了巡回上诉法院法官Ketanji Brown Jackson。她是我们国家顶尖的法律人才之一，将继续布雷耶大法官卓越的遗产。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.6596046090126038
一位曾在私人执业中担任顶尖诉讼律师的人。一位前联邦公设辩护人。来自一个公共学校教育工作者和警察的家庭。一个共识的建立者。自从她被提名以来，她得到了广泛的支持——从警察兄弟会到民主党和共和党任命的前法官。

如果我们要推进自由与正义，我们需要保卫边界并修复移民系统。

我们可以做到这两点。在我们的边界，我们安装了新技术，比如尖端扫描仪，以更好地检测毒品走私。

我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人贩子。

我们正在设立专门的移民法官，以便逃离迫害和暴力的家庭能够更快地听取他们的案件。

我们正在确保与南美和中美洲的合作伙伴达成承诺，以接纳更多的难民并保护他们自己的边界。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.6597143411636353
一位曾在私人执业中担任顶尖诉讼律师的人。一位前联邦公设辩护人。来自一个公共学校教育工作者和警察的家庭。一个共识的建立者。自从她被提名以来，她得到了广泛的支持——从警察兄弟会到民主党和共和党任命的前法官。

如果我们要推进自由与正义，我们需要保卫边界并修复移民系统。

我们可以做到这两点。在我们的边界，我们安装了新技术，比如尖端扫描仪，以更好地检测毒品走私。

我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人贩子。

我们正在设立专门的移民法官，以便逃离迫害和暴力的家庭能够更快地听取他们的案件。

我们正在确保与南美和中美洲的合作伙伴达成承诺，以接纳更多的难民并保护他们自己的边界。
--------------------------------------------------------------------------------
```

## 最大边际相关性搜索 (MMR)
最大边际相关性优化查询的相似性和所选文档之间的多样性。

```python
docs_with_score = db.max_marginal_relevance_search_with_score(query)
```

```python
for doc, score in docs_with_score:
    print("-" * 80)
    print("Score: ", score)
    print(doc.page_content)
    print("-" * 80)
```
```output
--------------------------------------------------------------------------------
Score:  0.6077010035514832
今晚。我呼吁参议院：通过《投票自由法》。通过《约翰·刘易斯投票权法》。同时，通过《披露法》，让美国人知道谁在资助我们的选举。

今晚，我想表彰一位为这个国家奉献一生的人：大法官斯蒂芬·布雷耶——一位退伍军人、宪法学者，以及即将退休的美国最高法院大法官。布雷耶大法官，感谢您的服务。

总统最严肃的宪法责任之一就是提名某人担任美国最高法院大法官。

而我在4天前做到了这一点，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律人才之一，将继续布雷耶大法官卓越的遗产。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.6852865219116211
这将改变美国，使我们走上赢得与世界其他国家——特别是中国——的21世纪经济竞争的道路。

正如我告诉习近平的，那永远不是一个好的选择去打赌美国人民。

我们将为数百万美国人创造良好的就业机会，现代化全国各地的道路、机场、港口和水道。

我们将全力以赴抵御气候危机的破坏性影响，促进环境公正。

我们将建立一个全国范围内的500,000个电动车充电站的网络，开始更换有毒的铅管——让每个孩子和每个美国人在家和学校都能喝到干净的水，为每个美国人提供可负担的高速互联网——无论是城市、郊区、农村还是部落社区。

已经宣布了4,000个项目。

今晚，我宣布今年我们将开始修复超过65,000英里破损的高速公路和1,500座桥梁。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.6866700053215027
我们无法改变我们曾经的分裂。但我们可以改变前进的方式——在COVID-19和我们必须共同面对的其他问题上。

我最近在纽约市警察局访问，正值维尔伯特·莫拉警官和他的搭档杰森·里维拉警官的葬礼之后几天。

他们在响应911电话时，被一名持有被盗枪支的男子射杀。

莫拉警官27岁。

里维拉警官22岁。

两位多米尼加裔美国人，他们在后来选择作为警察巡逻的同一条街上长大。

我与他们的家人交谈，并告诉他们我们永远欠他们的牺牲一份债务，我们将继续他们的使命，以恢复每个社区应得的信任和安全。

我在这些问题上工作了很长时间。

我知道什么有效：投资于犯罪预防和社区警察，他们将巡逻，了解邻里，并能够恢复信任和安全。
--------------------------------------------------------------------------------
--------------------------------------------------------------------------------
Score:  0.6936529278755188
但由于长期接触燃烧坑，癌症摧毁了希思的肺部和身体。

丹妮尔说希思一直是个斗士，直到最后。

他不知道如何停止战斗，她也一样。

通过她的痛苦，她找到了要求我们做得更好的目标。

今晚，丹妮尔——我们正在这样做。

退伍军人事务部正在开创将有毒暴露与疾病联系起来的新方法，已经帮助更多退伍军人获得福利。

今晚，我宣布我们将扩大对九种呼吸系统癌症患者的退伍军人资格的认可。

我还呼吁国会：通过一项法律，确保在伊拉克和阿富汗遭受有毒暴露的退伍军人最终获得他们应得的福利和全面的医疗保健。

第四，让我们终结癌症，如今所知的癌症。

这对我和吉尔、对卡马拉，以及对你们许多人来说都是个人的。

癌症是美国第二大死亡原因，仅次于心脏病。
--------------------------------------------------------------------------------
```

## 使用向量存储

上面，我们从头创建了一个向量存储。然而，我们通常希望使用现有的向量存储。为了做到这一点，我们可以直接初始化它。

```python
store = Kinetica(
    collection_name=COLLECTION_NAME,
    config=connection,
    embedding_function=embeddings,
)
```

### 添加文档
我们可以将文档添加到现有的向量存储中。

```python
store.add_documents([Document(page_content="foo")])
```

```output
['b94dc67c-ce7e-11ee-b8cb-b940b0e45762']
```

```python
docs_with_score = db.similarity_search_with_score("foo")
```

```python
docs_with_score[0]
```

```output
(Document(page_content='foo'), 0.0)
```

```python
docs_with_score[1]
```

```output
(Document(page_content='A former top litigator in private practice. A former federal public defender. And from a family of public school educators and police officers. A consensus builder. Since she’s been nominated, she’s received a broad range of support—from the Fraternal Order of Police to former judges appointed by Democrats and Republicans. \n\nAnd if we are to advance liberty and justice, we need to secure the Border and fix the immigration system. \n\nWe can do both. At our border, we’ve installed new technology like cutting-edge scanners to better detect drug smuggling.  \n\nWe’ve set up joint patrols with Mexico and Guatemala to catch more human traffickers.  \n\nWe’re putting in place dedicated immigration judges so families fleeing persecution and violence can have their cases heard faster. \n\nWe’re securing commitments and supporting partners in South and Central America to host more refugees and secure their own borders.', metadata={'source': '../../how_to/state_of_the_union.txt'}),
 0.6946534514427185)
```

### 覆盖向量存储

如果您有一个现有的集合，您可以通过执行 `from_documents` 并将 `pre_delete_collection` 设置为 True 来覆盖它。

```python
db = Kinetica.from_documents(
    documents=docs,
    embedding=embeddings,
    collection_name=COLLECTION_NAME,
    config=connection,
    pre_delete_collection=True,
)
```

```python
docs_with_score = db.similarity_search_with_score("foo")
```

```python
docs_with_score[0]
```

```output
(Document(page_content='A former top litigator in private practice. A former federal public defender. And from a family of public school educators and police officers. A consensus builder. Since she’s been nominated, she’s received a broad range of support—from the Fraternal Order of Police to former judges appointed by Democrats and Republicans. \n\nAnd if we are to advance liberty and justice, we need to secure the Border and fix the immigration system. \n\nWe can do both. At our border, we’ve installed new technology like cutting-edge scanners to better detect drug smuggling.  \n\nWe’ve set up joint patrols with Mexico and Guatemala to catch more human traffickers.  \n\nWe’re putting in place dedicated immigration judges so families fleeing persecution and violence can have their cases heard faster. \n\nWe’re securing commitments and supporting partners in South and Central America to host more refugees and secure their own borders.', metadata={'source': '../../how_to/state_of_the_union.txt'}),
 0.6946534514427185)
```

### 使用 VectorStore 作为检索器


```python
retriever = store.as_retriever()
```


```python
print(retriever)
```
```output
tags=['Kinetica', 'OpenAIEmbeddings'] vectorstore=<langchain_community.vectorstores.kinetica.Kinetica object at 0x7f1644375e20>
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)