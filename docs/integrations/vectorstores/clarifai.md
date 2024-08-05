---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/clarifai.ipynb
---

# Clarifai

>[Clarifai](https://www.clarifai.com/) 是一个人工智能平台，提供完整的人工智能生命周期，包括数据探索、数据标注、模型训练、评估和推理。上传输入后，Clarifai 应用可以用作向量数据库。

本笔记本展示了如何使用与 `Clarifai` 向量数据库相关的功能。示例展示了文本语义搜索的能力。Clarifai 还支持图像、视频帧的语义搜索，以及本地化搜索（请参见 [Rank](https://docs.clarifai.com/api-guide/search/rank)）和属性搜索（请参见 [Filter](https://docs.clarifai.com/api-guide/search/filter)）。

要使用 Clarifai，您必须拥有一个账户和个人访问令牌（PAT）密钥。
[在这里查看](https://clarifai.com/settings/security)以获取或创建 PAT。

# 依赖项


```python
# Install required dependencies
%pip install --upgrade --quiet  clarifai langchain-community
```

# 导入
在这里我们将设置个人访问令牌。您可以在平台的设置/安全性中找到您的PAT。

```python
# Please login and get your API key from  https://clarifai.com/settings/security
from getpass import getpass

CLARIFAI_PAT = getpass()
```
```output
 ········
```

```python
# Import the required modules
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import Clarifai
from langchain_text_splitters import CharacterTextSplitter
```

# 设置
设置用户 ID 和应用程序 ID，以便上传文本数据。注意：创建该应用程序时，请选择适合索引文本文档的基础工作流，例如语言理解工作流。

您需要首先在 [Clarifai](https://clarifai.com/login) 上创建一个帐户，然后创建一个应用程序。

```python
USER_ID = "USERNAME_ID"
APP_ID = "APPLICATION_ID"
NUMBER_OF_DOCS = 2
```

## 从文本创建
从文本列表创建一个 Clarifai 向量存储。本节将每个文本及其相应的元数据上传到 Clarifai 应用程序。然后可以使用 Clarifai 应用程序进行语义搜索，以查找相关文本。

```python
texts = [
    "I really enjoy spending time with you",
    "I hate spending time with my dog",
    "I want to go for a run",
    "I went to the movies yesterday",
    "I love playing soccer with my friends",
]

metadatas = [
    {"id": i, "text": text, "source": "book 1", "category": ["books", "modern"]}
    for i, text in enumerate(texts)
]
```

或者，您可以选择为输入提供自定义输入 ID。

```python
idlist = ["text1", "text2", "text3", "text4", "text5"]
metadatas = [
    {"id": idlist[i], "text": text, "source": "book 1", "category": ["books", "modern"]}
    for i, text in enumerate(texts)
]
```

```python
# 可以通过将 pat 作为参数初始化 clarifai 向量存储！
clarifai_vector_db = Clarifai(
    user_id=USER_ID,
    app_id=APP_ID,
    number_of_docs=NUMBER_OF_DOCS,
)
```

将数据上传到 clarifai 应用程序。

```python
# 带元数据和自定义输入 ID 上传。
response = clarifai_vector_db.add_texts(texts=texts, ids=idlist, metadatas=metadatas)

# 不带元数据上传（不推荐）- 因为您将无法根据元数据执行搜索操作。
# 自定义 input_id（可选）
response = clarifai_vector_db.add_texts(texts=texts)
```

您可以通过以下方式创建一个 clarifai 向量数据库存储，并直接将所有输入导入到您的应用程序中，

```python
clarifai_vector_db = Clarifai.from_texts(
    user_id=USER_ID,
    app_id=APP_ID,
    texts=texts,
    metadatas=metadatas,
)
```

使用相似性搜索功能搜索相似文本。

```python
docs = clarifai_vector_db.similarity_search("I would like to see you")
docs
```

```output
[Document(page_content='I really enjoy spending time with you', metadata={'text': 'I really enjoy spending time with you', 'id': 'text1', 'source': 'book 1', 'category': ['books', 'modern']})]
```

此外，您可以通过元数据过滤搜索结果。

```python
# 您可以通过利用元数据过滤器在应用程序中进行强大的过滤。
# 这将限制相似性查询，仅限于具有 "source" 键且值匹配 "book 1" 的文本
book1_similar_docs = clarifai_vector_db.similarity_search(
    "I would love to see you", filter={"source": "book 1"}
)

# 您还可以在输入的元数据中使用列表，然后选择与列表中的某个项匹配的内容。这对于以下类别非常有用：
book_category_similar_docs = clarifai_vector_db.similarity_search(
    "I would love to see you", filter={"category": ["books"]}
)
```

## 从文档中
从文档列表中创建一个 Clarifai 向量存储。此部分将上传每个文档及其相应的元数据到 Clarifai 应用程序。然后可以使用 Clarifai 应用程序进行语义搜索，以找到相关文档。

```python
loader = TextLoader("your_local_file_path.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
```

```python
USER_ID = "USERNAME_ID"
APP_ID = "APPLICATION_ID"
NUMBER_OF_DOCS = 4
```

创建一个 clarifai 向量数据库类，并将所有文档导入 clarifai 应用程序。

```python
clarifai_vector_db = Clarifai.from_documents(
    user_id=USER_ID,
    app_id=APP_ID,
    documents=docs,
    number_of_docs=NUMBER_OF_DOCS,
)
```

```python
docs = clarifai_vector_db.similarity_search("与人口相关的文本")
docs
```

## 从现有应用程序
在Clarifai中，我们拥有出色的工具，通过API或UI向应用程序（即项目）添加数据。大多数用户在与LangChain互动之前，已经完成了这一过程，因此本示例将使用现有应用程序中的数据来进行搜索。请查看我们的[API文档](https://docs.clarifai.com/api-guide/data/create-get-update-delete)和[UI文档](https://docs.clarifai.com/portal-guide/data)。然后，可以使用Clarifai应用程序进行语义搜索，以查找相关文档。


```python
USER_ID = "USERNAME_ID"
APP_ID = "APPLICATION_ID"
NUMBER_OF_DOCS = 4
```


```python
clarifai_vector_db = Clarifai(
    user_id=USER_ID,
    app_id=APP_ID,
    number_of_docs=NUMBER_OF_DOCS,
)
```


```python
docs = clarifai_vector_db.similarity_search(
    "Texts related to ammuniction and president wilson"
)
```


```python
docs[0].page_content
```



```output
"President Wilson, generally acclaimed as the leader of the world's democracies,\nphrased for civilization the arguments against autocracy in the great peace conference\nafter the war. The President headed the American delegation to that conclave of world\nre-construction. With him as delegates to the conference were Robert Lansing, Secretary\nof State; Henry White, former Ambassador to France and Italy; Edward M. House and\nGeneral Tasker H. Bliss.\nRepresenting American Labor at the International Labor conference held in Paris\nsimultaneously with the Peace Conference were Samuel Gompers, president of the\nAmerican Federation of Labor; William Green, secretary-treasurer of the United Mine\nWorkers of America; John R. Alpine, president of the Plumbers' Union; James Duncan,\npresident of the International Association of Granite Cutters; Frank Duffy, president of\nthe United Brotherhood of Carpenters and Joiners, and Frank Morrison, secretary of the\nAmerican Federation of Labor.\nEstimating the share of each Allied nation in the great victory, mankind will\nconclude that the heaviest cost in proportion to prewar population and treasure was paid\nby the nations that first felt the shock of war, Belgium, Serbia, Poland and France. All\nfour were the battle-grounds of huge armies, oscillating in a bloody frenzy over once\nfertile fields and once prosperous towns.\nBelgium, with a population of 8,000,000, had a casualty list of more than 350,000;\nFrance, with its casualties of 4,000,000 out of a population (including its colonies) of\n90,000,000, is really the martyr nation of the world. Her gallant poilus showed the world\nhow cheerfully men may die in defense of home and liberty. Huge Russia, including\nhapless Poland, had a casualty list of 7,000,000 out of its entire population of\n180,000,000. The United States out of a population of 110,000,000 had a casualty list of\n236,117 for nineteen months of war; of these 53,169 were killed or died of disease;\n179,625 were wounded; and 3,323 prisoners or missing."
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)