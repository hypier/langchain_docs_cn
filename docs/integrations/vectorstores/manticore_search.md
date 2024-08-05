---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/manticore_search.ipynb
---

# ManticoreSearch 向量存储

[ManticoreSearch](https://manticoresearch.com/) 是一个开源搜索引擎，提供快速、可扩展和用户友好的功能。它起源于 [Sphinx Search](http://sphinxsearch.com/) 的一个分支，经过发展，融入了现代搜索引擎的特性和改进。ManticoreSearch 以其强大的性能和易于集成到各种应用程序中的特点而脱颖而出。

ManticoreSearch 最近推出了 [向量搜索功能](https://manual.manticoresearch.com/dev/Searching/KNN)，从搜索引擎版本 6.2 开始，仅在安装了 [manticore-columnar-lib](https://github.com/manticoresoftware/columnar) 包的情况下可用。此功能是一个重要的进展，允许基于向量相似性执行搜索。

截至目前，向量搜索功能仅在搜索引擎的开发（dev）版本中可用。因此，必须使用开发版的 [manticoresearch-dev](https://pypi.org/project/manticoresearch-dev/) Python 客户端，以有效利用此功能。

## 设置环境

启动带有 ManticoreSearch 的 Docker 容器并安装 manticore-columnar-lib 包（可选）


```python
import time

# 启动容器
containers = !docker ps --filter "name=langchain-manticoresearch-server" -q
if len(containers) == 0:
    !docker run -d -p 9308:9308 --name langchain-manticoresearch-server manticoresearch/manticore:dev
    time.sleep(20)  # 等待容器启动

# 获取容器 ID
container_id = containers[0]

# 以 root 用户身份安装 manticore-columnar-lib 包
!docker exec -it --user 0 {container_id} apt-get update
!docker exec -it --user 0 {container_id} apt-get install -y manticore-columnar-lib

# 重启容器
!docker restart {container_id}
```
```output
Get:1 http://repo.manticoresearch.com/repository/manticoresearch_jammy_dev jammy InRelease [3525 kB]
Get:2 http://archive.ubuntu.com/ubuntu jammy InRelease [270 kB]            
Get:3 http://security.ubuntu.com/ubuntu jammy-security InRelease [110 kB]      
Get:4 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [119 kB]        
Get:5 http://security.ubuntu.com/ubuntu jammy-security/universe amd64 Packages [1074 kB]
Get:6 http://archive.ubuntu.com/ubuntu jammy-backports InRelease [109 kB]      
Get:7 http://archive.ubuntu.com/ubuntu jammy/universe amd64 Packages [17.5 MB] 
Get:8 http://security.ubuntu.com/ubuntu jammy-security/main amd64 Packages [1517 kB]
Get:9 http://security.ubuntu.com/ubuntu jammy-security/restricted amd64 Packages [1889 kB]
Get:10 http://security.ubuntu.com/ubuntu jammy-security/multiverse amd64 Packages [44.6 kB]
Get:11 http://archive.ubuntu.com/ubuntu jammy/restricted amd64 Packages [164 kB]
Get:12 http://archive.ubuntu.com/ubuntu jammy/multiverse amd64 Packages [266 kB]
Get:13 http://archive.ubuntu.com/ubuntu jammy/main amd64 Packages [1792 kB]    
Get:14 http://archive.ubuntu.com/ubuntu jammy-updates/multiverse amd64 Packages [50.4 kB]
Get:15 http://archive.ubuntu.com/ubuntu jammy-updates/restricted amd64 Packages [1927 kB]
Get:16 http://archive.ubuntu.com/ubuntu jammy-updates/universe amd64 Packages [1346 kB]
Get:17 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 Packages [1796 kB]
Get:18 http://archive.ubuntu.com/ubuntu jammy-backports/universe amd64 Packages [28.1 kB]
Get:19 http://archive.ubuntu.com/ubuntu jammy-backports/main amd64 Packages [50.4 kB]
Get:20 http://repo.manticoresearch.com/repository/manticoresearch_jammy_dev jammy/main amd64 Packages [5020 kB]
Fetched 38.6 MB in 7s (5847 kB/s)                                              
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  manticore-columnar-lib
0 upgraded, 1 newly installed, 0 to remove and 21 not upgraded.
Need to get 1990 kB of archives.
After this operation, 10.0 MB of additional disk space will be used.
Get:1 http://repo.manticoresearch.com/repository/manticoresearch_jammy_dev jammy/main amd64 manticore-columnar-lib amd64 2.2.5-240217-a5342a1 [1990 kB]
Fetched 1990 kB in 1s (1505 kB/s)                 
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package manticore-columnar-lib.
(Reading database ... 12260 files and directories currently installed.)
Preparing to unpack .../manticore-columnar-lib_2.2.5-240217-a5342a1_amd64.deb ...
Unpacking manticore-columnar-lib (2.2.5-240217-a5342a1) ...
Setting up manticore-columnar-lib (2.2.5-240217-a5342a1) ...
a546aec22291
```
安装 ManticoreSearch Python 客户端


```python
%pip install --upgrade --quiet manticoresearch-dev
```
```output

[1m[[0m[34;49mnotice[0m[1;39;49m][0m[39;49m A new release of pip is available: [0m[31;49m23.2.1[0m[39;49m -> [0m[32;49m24.0[0m
[1m[[0m[34;49mnotice[0m[1;39;49m][0m[39;49m To update, run: [0m[32;49mpip install --upgrade pip[0m
Note: you may need to restart the kernel to use updated packages.
```
我们希望使用 OpenAIEmbeddings，因此我们需要获取 OpenAI API 密钥。


```python
from langchain.text_splitter import CharacterTextSplitter
from langchain_community.embeddings import GPT4AllEmbeddings
from langchain_community.vectorstores import ManticoreSearch, ManticoreSearchSettings
```


```python
from langchain_community.document_loaders import TextLoader

loader = TextLoader("../../modules/paul_graham_essay.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=100, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = GPT4AllEmbeddings()
```


```python
for d in docs:
    d.metadata = {"some": "metadata"}
settings = ManticoreSearchSettings(table="manticoresearch_vector_search_example")
docsearch = ManticoreSearch.from_documents(docs, embeddings, config=settings)

query = "Robert Morris is"
docs = docsearch.similarity_search(query)
print(docs)
```
```output
[Document(page_content='Computer Science is an uneasy alliance between two halves, theory and systems. The theory people prove things, and the systems people build things. I wanted to build things. I had plenty of respect for theory — indeed, a sneaking suspicion that it was the more admirable of the two halves — but building things seemed so much more exciting.', metadata={'some': 'metadata'}), Document(page_content="I applied to 3 grad schools: MIT and Yale, which were renowned for AI at the time, and Harvard, which I'd visited because Rich Draves went there, and was also home to Bill Woods, who'd invented the type of parser I used in my SHRDLU clone. Only Harvard accepted me, so that was where I went.", metadata={'some': 'metadata'}), Document(page_content='For my undergraduate thesis, I reverse-engineered SHRDLU. My God did I love working on that program. It was a pleasing bit of code, but what made it even more exciting was my belief — hard to imagine now, but not unique in 1985 — that it was already climbing the lower slopes of intelligence.', metadata={'some': 'metadata'}), Document(page_content="The problem with systems work, though, was that it didn't last. Any program you wrote today, no matter how good, would be obsolete in a couple decades at best. People might mention your software in footnotes, but no one would actually use it. And indeed, it would seem very feeble work. Only people with a sense of the history of the field would even realize that, in its time, it had been good.", metadata={'some': 'metadata'})]
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)