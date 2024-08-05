---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/oracle.ipynb
---

# Oracle AI 向量搜索：向量存储

Oracle AI 向量搜索旨在处理人工智能（AI）工作负载，它允许您基于语义而非关键词查询数据。
Oracle AI 向量搜索的最大好处之一是，可以将非结构化数据的语义搜索与业务数据的关系搜索结合在一个系统中。
这不仅强大，而且显著更有效，因为您无需添加专门的向量数据库，从而消除了多个系统之间数据碎片化的痛苦。

此外，您的向量可以受益于 Oracle 数据库的所有强大功能，例如：

 * [分区支持](https://www.oracle.com/database/technologies/partitioning.html)
 * [真实应用集群可扩展性](https://www.oracle.com/database/real-application-clusters/)
 * [Exadata 智能扫描](https://www.oracle.com/database/technologies/exadata/software/smartscan/)
 * [跨地理分布数据库的分片处理](https://www.oracle.com/database/distributed-database/)
 * [事务](https://docs.oracle.com/en/database/oracle/oracle-database/23/cncpt/transactions.html)
 * [并行 SQL](https://docs.oracle.com/en/database/oracle/oracle-database/21/vldbg/parallel-exec-intro.html#GUID-D28717E4-0F77-44F5-BB4E-234C31D4E4BA)
 * [灾难恢复](https://www.oracle.com/database/data-guard/)
 * [安全性](https://www.oracle.com/security/database-security/)
 * [Oracle 机器学习](https://www.oracle.com/artificial-intelligence/database-machine-learning/)
 * [Oracle 图形数据库](https://www.oracle.com/database/integrated-graph-database/)
 * [Oracle 空间和图形](https://www.oracle.com/database/spatial/)
 * [Oracle 区块链](https://docs.oracle.com/en/database/oracle/oracle-database/23/arpls/dbms_blockchain_table.html#GUID-B469E277-978E-4378-A8C1-26D3FF96C9A6)
 * [JSON](https://docs.oracle.com/en/database/oracle/oracle-database/23/adjsn/json-in-oracle-database.html)

如果您刚开始使用 Oracle 数据库，建议您探索 [免费的 Oracle 23 AI](https://www.oracle.com/database/free/#resources)，它提供了设置数据库环境的良好介绍。在使用数据库时，通常建议避免默认使用系统用户；相反，您可以创建自己的用户以增强安全性和自定义。有关用户创建的详细步骤，请参阅我们的 [端到端指南](https://github.com/langchain-ai/langchain/blob/master/cookbook/oracleai_demo.ipynb)，该指南还展示了如何在 Oracle 中设置用户。此外，理解用户权限对于有效管理数据库安全性至关重要。您可以在官方 [Oracle 指南](https://docs.oracle.com/en/database/oracle/oracle-database/19/admqs/administering-user-accounts-and-security.html#GUID-36B21D72-1BBB-46C9-A0C9-F0D2A8591B8D) 中了解更多关于管理用户账户和安全性的主题。

### 使用 Langchain 与 Oracle AI 向量搜索的先决条件

您需要通过 `pip install -qU langchain-community` 安装 `langchain-community` 以使用此集成。

请安装 Oracle Python 客户端驱动程序，以便将 Langchain 与 Oracle AI 向量搜索结合使用。

```python
# pip install oracledb
```

### 连接到 Oracle AI 向量搜索

以下示例代码将展示如何连接到 Oracle 数据库。默认情况下，python-oracledb 以“Thin”模式运行，该模式直接连接到 Oracle 数据库。此模式不需要 Oracle 客户端库。然而，当 python-oracledb 使用这些库时，某些附加功能可用。当使用 Oracle 客户端库时，python-oracledb 被称为处于“Thick”模式。这两种模式都具有全面的功能，支持 Python 数据库 API v2.0 规范。请参阅以下 [指南](https://python-oracledb.readthedocs.io/en/latest/user_guide/appendix_a.html#featuresummary)，了解每种模式支持的功能。如果您无法使用 thin-mode，您可能想切换到 thick-mode。

```python
import oracledb

username = "username"
password = "password"
dsn = "ipaddress:port/orclpdb1"

try:
    connection = oracledb.connect(user=username, password=password, dsn=dsn)
    print("Connection successful!")
except Exception as e:
    print("Connection failed!")
```

### 导入使用 Oracle AI 向量搜索所需的依赖项


```python
from langchain_community.vectorstores import oraclevs
from langchain_community.vectorstores.oraclevs import OracleVS
from langchain_community.vectorstores.utils import DistanceStrategy
from langchain_core.documents import Document
from langchain_huggingface import HuggingFaceEmbeddings
```

### 加载文档


```python
# Define a list of documents (The examples below are 5 random documents from Oracle Concepts Manual )

documents_json_list = [
    {
        "id": "cncpt_15.5.3.2.2_P4",
        "text": "如果任何前面问题的回答是肯定的，那么数据库将停止搜索并从指定的表空间分配空间；否则，空间将从数据库默认的共享临时表空间中分配。",
        "link": "https://docs.oracle.com/en/database/oracle/oracle-database/23/cncpt/logical-storage-structures.html#GUID-5387D7B2-C0CA-4C1E-811B-C7EB9B636442",
    },
    {
        "id": "cncpt_15.5.5_P1",
        "text": "在数据库打开时，表空间可以是在线（可访问）或离线（不可访问）。\n表空间通常是在线的，以便其数据对用户可用。SYSTEM 表空间和临时表空间不能被设置为离线。",
        "link": "https://docs.oracle.com/en/database/oracle/oracle-database/23/cncpt/logical-storage-structures.html#GUID-D02B2220-E6F5-40D9-AFB5-BC69BCEF6CD4",
    },
    {
        "id": "cncpt_22.3.4.3.1_P2",
        "text": "数据库以不同于其他数据类型的方式存储 LOB。创建 LOB 列隐式地创建了一个 LOB 段和一个 LOB 索引。包含 LOB 段和 LOB 索引的表空间（它们总是一起存储）可能与包含表的表空间不同。\n有时，数据库可以在表本身中存储少量 LOB 数据，而不是在单独的 LOB 段中。",
        "link": "https://docs.oracle.com/en/database/oracle/oracle-database/23/cncpt/concepts-for-database-developers.html#GUID-3C50EAB8-FC39-4BB3-B680-4EACCE49E866",
    },
    {
        "id": "cncpt_22.3.4.3.1_P3",
        "text": "LOB 段以称为块的片段存储数据。块是一组逻辑上连续的数据块，是 LOB 的最小分配单元。表中的一行存储一个称为 LOB 定位符的指针，该指针指向 LOB 索引。当查询表时，数据库使用 LOB 索引快速定位 LOB 块。",
        "link": "https://docs.oracle.com/en/database/oracle/oracle-database/23/cncpt/concepts-for-database-developers.html#GUID-3C50EAB8-FC39-4BB3-B680-4EACCE49E866",
    },
]
```


```python
# Create Langchain Documents

documents_langchain = []

for doc in documents_json_list:
    metadata = {"id": doc["id"], "link": doc["link"]}
    doc_langchain = Document(page_content=doc["text"], metadata=metadata)
    documents_langchain.append(doc_langchain)
```

### 使用AI向量搜索创建具有不同距离度量的向量存储

首先，我们将创建三个向量存储，每个存储使用不同的距离函数。由于我们尚未在其中创建索引，因此它们现在只会创建表。稍后我们将使用这些向量存储来创建HNSW索引。要了解Oracle AI向量搜索支持的不同类型的索引，请参考以下[指南](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/manage-different-categories-vector-indexes.html)。

您可以手动连接到Oracle数据库，将看到三个表：
Documents_DOT、Documents_COSINE和Documents_EUCLIDEAN。

然后我们将创建三个额外的表Documents_DOT_IVF、Documents_COSINE_IVF和Documents_EUCLIDEAN_IVF，这些表将用于在表上创建IVF索引，而不是HNSW索引。

```python
# Ingest documents into Oracle Vector Store using different distance strategies

# When using our API calls, start by initializing your vector store with a subset of your documents
# through from_documents(), then incrementally add more documents using add_texts().
# This approach prevents system overload and ensures efficient document processing.

model = HuggingFaceEmbeddings(model_name="sentence-transformers/all-mpnet-base-v2")

vector_store_dot = OracleVS.from_documents(
    documents_langchain,
    model,
    client=connection,
    table_name="Documents_DOT",
    distance_strategy=DistanceStrategy.DOT_PRODUCT,
)
vector_store_max = OracleVS.from_documents(
    documents_langchain,
    model,
    client=connection,
    table_name="Documents_COSINE",
    distance_strategy=DistanceStrategy.COSINE,
)
vector_store_euclidean = OracleVS.from_documents(
    documents_langchain,
    model,
    client=connection,
    table_name="Documents_EUCLIDEAN",
    distance_strategy=DistanceStrategy.EUCLIDEAN_DISTANCE,
)

# Ingest documents into Oracle Vector Store using different distance strategies
vector_store_dot_ivf = OracleVS.from_documents(
    documents_langchain,
    model,
    client=connection,
    table_name="Documents_DOT_IVF",
    distance_strategy=DistanceStrategy.DOT_PRODUCT,
)
vector_store_max_ivf = OracleVS.from_documents(
    documents_langchain,
    model,
    client=connection,
    table_name="Documents_COSINE_IVF",
    distance_strategy=DistanceStrategy.COSINE,
)
vector_store_euclidean_ivf = OracleVS.from_documents(
    documents_langchain,
    model,
    client=connection,
    table_name="Documents_EUCLIDEAN_IVF",
    distance_strategy=DistanceStrategy.EUCLIDEAN_DISTANCE,
)
```

### 演示文本的添加和删除操作，以及基本的相似性搜索



```python
def manage_texts(vector_stores):
    """
    向每个向量存储添加文本，演示重复添加的错误处理，并执行文本的删除。展示每个向量存储的相似性搜索和索引创建。

    Args:
    - vector_stores (list): OracleVS 实例的列表。
    """
    texts = ["Rohan", "Shailendra"]
    metadata = [
        {"id": "100", "link": "文档示例测试 1"},
        {"id": "101", "link": "文档示例测试 2"},
    ]

    for i, vs in enumerate(vector_stores, start=1):
        # 添加文本
        try:
            vs.add_texts(texts, metadata)
            print(f"\n\n\n向量存储 {i} 的文本添加完成\n\n\n")
        except Exception as ex:
            print(f"\n\n\n向量存储 {i} 的重复添加预期错误\n\n\n")

        # 使用 'id' 的值删除文本
        vs.delete([metadata[0]["id"]])
        print(f"\n\n\n向量存储 {i} 的文本删除完成\n\n\n")

        # 相似性搜索
        results = vs.similarity_search("LOBs 在 Oracle 数据库中如何存储", 2)
        print(f"\n\n\n向量存储 {i} 的相似性搜索结果: {results}\n\n\n")


vector_store_list = [
    vector_store_dot,
    vector_store_max,
    vector_store_euclidean,
    vector_store_dot_ivf,
    vector_store_max_ivf,
    vector_store_euclidean_ivf,
]
manage_texts(vector_store_list)
```

### 演示使用特定参数为每种距离策略创建索引



```python
def create_search_indices(connection):
    """
    Creates search indices for the vector stores, each with specific parameters tailored to their distance strategy.
    """
    # Index for DOT_PRODUCT strategy
    # Notice we are creating a HNSW index with default parameters
    # This will default to creating a HNSW index with 8 Parallel Workers and use the Default Accuracy used by Oracle AI Vector Search
    oraclevs.create_index(
        connection,
        vector_store_dot,
        params={"idx_name": "hnsw_idx1", "idx_type": "HNSW"},
    )

    # Index for COSINE strategy with specific parameters
    # Notice we are creating a HNSW index with parallel 16 and Target Accuracy Specification as 97 percent
    oraclevs.create_index(
        connection,
        vector_store_max,
        params={
            "idx_name": "hnsw_idx2",
            "idx_type": "HNSW",
            "accuracy": 97,
            "parallel": 16,
        },
    )

    # Index for EUCLIDEAN_DISTANCE strategy with specific parameters
    # Notice we are creating a HNSW index by specifying Power User Parameters which are neighbors = 64 and efConstruction = 100
    oraclevs.create_index(
        connection,
        vector_store_euclidean,
        params={
            "idx_name": "hnsw_idx3",
            "idx_type": "HNSW",
            "neighbors": 64,
            "efConstruction": 100,
        },
    )

    # Index for DOT_PRODUCT strategy with specific parameters
    # Notice we are creating an IVF index with default parameters
    # This will default to creating an IVF index with 8 Parallel Workers and use the Default Accuracy used by Oracle AI Vector Search
    oraclevs.create_index(
        connection,
        vector_store_dot_ivf,
        params={
            "idx_name": "ivf_idx1",
            "idx_type": "IVF",
        },
    )

    # Index for COSINE strategy with specific parameters
    # Notice we are creating an IVF index with parallel 32 and Target Accuracy Specification as 90 percent
    oraclevs.create_index(
        connection,
        vector_store_max_ivf,
        params={
            "idx_name": "ivf_idx2",
            "idx_type": "IVF",
            "accuracy": 90,
            "parallel": 32,
        },
    )

    # Index for EUCLIDEAN_DISTANCE strategy with specific parameters
    # Notice we are creating an IVF index by specifying Power User Parameters which is neighbor_part = 64
    oraclevs.create_index(
        connection,
        vector_store_euclidean_ivf,
        params={"idx_name": "ivf_idx3", "idx_type": "IVF", "neighbor_part": 64},
    )

    print("Index creation complete.")


create_search_indices(connection)
```

### 演示在所有六个向量存储上进行高级搜索，包括和不包括属性过滤 – 使用过滤时，我们只选择文档 ID 101，其他不选


```python
# Conduct advanced searches after creating the indices
def conduct_advanced_searches(vector_stores):
    query = "How are LOBS stored in Oracle Database"
    # Constructing a filter for direct comparison against document metadata
    # This filter aims to include documents whose metadata 'id' is exactly '2'
    filter_criteria = {"id": ["101"]}  # Direct comparison filter

    for i, vs in enumerate(vector_stores, start=1):
        print(f"\n--- Vector Store {i} Advanced Searches ---")
        # Similarity search without a filter
        print("\nSimilarity search results without filter:")
        print(vs.similarity_search(query, 2))

        # Similarity search with a filter
        print("\nSimilarity search results with filter:")
        print(vs.similarity_search(query, 2, filter=filter_criteria))

        # Similarity search with relevance score
        print("\nSimilarity search with relevance score:")
        print(vs.similarity_search_with_score(query, 2))

        # Similarity search with relevance score with filter
        print("\nSimilarity search with relevance score with filter:")
        print(vs.similarity_search_with_score(query, 2, filter=filter_criteria))

        # Max marginal relevance search
        print("\nMax marginal relevance search results:")
        print(vs.max_marginal_relevance_search(query, 2, fetch_k=20, lambda_mult=0.5))

        # Max marginal relevance search with filter
        print("\nMax marginal relevance search results with filter:")
        print(
            vs.max_marginal_relevance_search(
                query, 2, fetch_k=20, lambda_mult=0.5, filter=filter_criteria
            )
        )


conduct_advanced_searches(vector_store_list)
```

### 端到端演示
请参考我们的完整演示指南 [Oracle AI Vector Search 端到端演示指南](https://github.com/langchain-ai/langchain/tree/master/cookbook/oracleai_demo.ipynb)，以借助 Oracle AI Vector Search 构建一个端到端的 RAG 管道。

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)