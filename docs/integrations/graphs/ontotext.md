---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/graphs/ontotext.ipynb
---

# Ontotext GraphDB

>[Ontotext GraphDB](https://graphdb.ontotext.com/) 是一个图形数据库和知识发现工具，符合 [RDF](https://www.w3.org/RDF/) 和 [SPARQL](https://www.w3.org/TR/sparql11-query/) 标准。

>本笔记本展示了如何使用 LLM 提供自然语言查询（NLQ 到 SPARQL，也称为 `text2sparql`）功能，以供 `Ontotext GraphDB` 使用。

## GraphDB LLM 功能

`GraphDB` 支持一些 LLM 集成功能，如下所述 [here](https://github.com/w3c/sparql-dev/issues/193):

[gpt-queries](https://graphdb.ontotext.com/documentation/10.5/gpt-queries.html)

* 魔法谓词用于请求 LLM 提供文本、列表或表格，使用来自知识图谱 (KG) 的数据
* 查询解释
* 结果解释、总结、改写、翻译

[retrieval-graphdb-connector](https://graphdb.ontotext.com/documentation/10.5/retrieval-graphdb-connector.html)

* 在向量数据库中对 KG 实体进行索引
* 支持任何文本嵌入算法和向量数据库
* 使用与 GraphDB 为 Elastic、Solr、Lucene 使用的相同强大连接器（索引）语言
* 自动同步 RDF 数据的更改到 KG 实体索引
* 支持嵌套对象（GraphDB 版本 10.5 中没有 UI 支持）
* 将 KG 实体序列化为文本，如下所示（例如，对于一个葡萄酒数据集）：

```
Franvino:
- is a RedWine.
- made from grape Merlo.
- made from grape Cabernet Franc.
- has sugar dry.
- has year 2012.
```

[talk-to-graph](https://graphdb.ontotext.com/documentation/10.5/talk-to-graph.html)

* 使用定义的 KG 实体索引的简单聊天机器人


在本教程中，我们将不使用 GraphDB LLM 集成，而是使用 NLQ 生成 `SPARQL`。我们将使用 `Star Wars API` (`SWAPI`) 本体和数据集，您可以在 [here](https://github.com/Ontotext-AD/langchain-graphdb-qa-chain-demo/blob/main/starwars-data.trig) 查看。

## 设置

您需要一个正在运行的 GraphDB 实例。本教程展示了如何使用 [GraphDB Docker 镜像](https://hub.docker.com/r/ontotext/graphdb) 在本地运行数据库。它提供了一个 Docker Compose 配置，能够使用《星球大战》数据集填充 GraphDB。所有必要的文件，包括此笔记本，可以从 [GitHub 仓库 langchain-graphdb-qa-chain-demo](https://github.com/Ontotext-AD/langchain-graphdb-qa-chain-demo) 下载。

* 安装 [Docker](https://docs.docker.com/get-docker/)。本教程使用的是 Docker 版本 `24.0.7`，其中包含 [Docker Compose](https://docs.docker.com/compose/)。对于早期的 Docker 版本，您可能需要单独安装 Docker Compose。
* 在您机器上的本地文件夹中克隆 [GitHub 仓库 langchain-graphdb-qa-chain-demo](https://github.com/Ontotext-AD/langchain-graphdb-qa-chain-demo)。
* 使用以下脚本从同一文件夹启动 GraphDB
  
```
docker build --tag graphdb .
docker compose up -d graphdb
```

  您需要等待几秒钟，直到数据库在 `http://localhost:7200/` 上启动。《星球大战》数据集 `starwars-data.trig` 会自动加载到 `langchain` 仓库中。可以使用本地 SPARQL 端点 `http://localhost:7200/repositories/langchain` 运行查询。您还可以从您喜欢的网页浏览器打开 GraphDB 工作台 `http://localhost:7200/sparql`，在其中可以交互式地进行查询。
* 设置工作环境

如果您使用 `conda`，请创建并激活一个新的 conda 环境（例如 `conda create -n graph_ontotext_graphdb_qa python=3.9.18`）。

安装以下库：

```
pip install jupyter==1.0.0
pip install openai==1.6.1
pip install rdflib==7.0.0
pip install langchain-openai==0.0.2
pip install langchain>=0.1.5
```

使用以下命令运行 Jupyter：
```
jupyter notebook
```

## 指定本体

为了让 LLM 能够生成 SPARQL，它需要知道知识图谱的架构（本体）。可以通过 `OntotextGraphDBGraph` 类的两个参数之一来提供：

* `query_ontology`：在 SPARQL 端点上执行的 `CONSTRUCT` 查询，返回 KG 架构语句。我们建议将本体存储在自己的命名图中，这样可以更容易地获取相关语句（如下例所示）。不支持 `DESCRIBE` 查询，因为 `DESCRIBE` 返回对称简明边界描述（SCBD），即也包括传入的类链接。在具有百万实例的大型图中，这样做效率不高。请查看 https://github.com/eclipse-rdf4j/rdf4j/issues/4857
* `local_file`：本地 RDF 本体文件。支持的 RDF 格式有 `Turtle`、`RDF/XML`、`JSON-LD`、`N-Triples`、`Notation-3`、`Trig`、`Trix`、`N-Quads`。

在任何情况下，本体转储应：

* 包含关于类、属性、属性与类的关联（使用 rdfs:domain、schema:domainIncludes 或 OWL 限制）以及分类法（重要个体）的足够信息。
* 不包含过于冗长和无关的定义和示例，这些内容对 SPARQL 构建没有帮助。


```python
from langchain_community.graphs import OntotextGraphDBGraph

# feeding the schema using a user construct query

graph = OntotextGraphDBGraph(
    query_endpoint="http://localhost:7200/repositories/langchain",
    query_ontology="CONSTRUCT {?s ?p ?o} FROM <https://swapi.co/ontology/> WHERE {?s ?p ?o}",
)
```


```python
# feeding the schema using a local RDF file

graph = OntotextGraphDBGraph(
    query_endpoint="http://localhost:7200/repositories/langchain",
    local_file="/path/to/langchain_graphdb_tutorial/starwars-ontology.nt",  # change the path here
)
```

无论哪种方式，本体（架构）以 `Turtle` 格式提供给 LLM，因为带有适当前缀的 `Turtle` 是最紧凑且最容易让 LLM 记住的。

《星球大战》的本体有点不寻常，因为它包含了很多关于类的具体三元组，例如物种 `:Aleena` 生活在 `<planet/38>`，它们是 `:Reptile` 的子类，具有某些典型特征（平均身高、平均寿命、皮肤颜色），并且特定个体（角色）是该类的代表：


```
@prefix : <https://swapi.co/vocabulary/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

:Aleena a owl:Class, :Species ;
    rdfs:label "Aleena" ;
    rdfs:isDefinedBy <https://swapi.co/ontology/> ;
    rdfs:subClassOf :Reptile, :Sentient ;
    :averageHeight 80.0 ;
    :averageLifespan "79" ;
    :character <https://swapi.co/resource/aleena/47> ;
    :film <https://swapi.co/resource/film/4> ;
    :language "Aleena" ;
    :planet <https://swapi.co/resource/planet/38> ;
    :skinColor "blue", "gray" .

    ...

 ```


为了保持本教程的简单性，我们使用未加密的 GraphDB。如果 GraphDB 是加密的，您应该在初始化 `OntotextGraphDBGraph` 之前设置环境变量 'GRAPHDB_USERNAME' 和 'GRAPHDB_PASSWORD'。

```python
os.environ["GRAPHDB_USERNAME"] = "graphdb-user"
os.environ["GRAPHDB_PASSWORD"] = "graphdb-password"

graph = OntotextGraphDBGraph(
    query_endpoint=...,
    query_ontology=...
)
```

## 针对 StarWars 数据集的问题回答

我们现在可以使用 `OntotextGraphDBQAChain` 来问一些问题。


```python
import os

from langchain.chains import OntotextGraphDBQAChain
from langchain_openai import ChatOpenAI

# 我们将使用一个需要 OpenAI API 密钥的 OpenAI 模型。
# 不过，也可以使用其他模型：
# https://python.langchain.com/docs/integrations/chat/

# 将环境变量 `OPENAI_API_KEY` 设置为您的 OpenAI API 密钥
os.environ["OPENAI_API_KEY"] = "sk-***"

# 这里可以使用任何可用的 OpenAI 模型。
# 我们使用 'gpt-4-1106-preview' 是因为它具有更大的上下文窗口。
# 'gpt-4-1106-preview' 模型名称将来会被弃用，并将更改为 'gpt-4-turbo' 或类似名称，
# 因此请务必咨询 OpenAI API https://platform.openai.com/docs/models 以获取正确的名称。

chain = OntotextGraphDBQAChain.from_llm(
    ChatOpenAI(temperature=0, model_name="gpt-4-1106-preview"),
    graph=graph,
    verbose=True,
)
```

让我们问一个简单的问题。


```python
chain.invoke({chain.input_key: "What is the climate on Tatooine?"})[chain.output_key]
```
```output


[1m> Entering new OntotextGraphDBQAChain chain...[0m
Generated SPARQL:
[32;1m[1;3mPREFIX : <https://swapi.co/vocabulary/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?climate
WHERE {
  ?planet rdfs:label "Tatooine" ;
          :climate ?climate .
}[0m

[1m> Finished chain.[0m
```


```output
'塔图因的气候是干旱的。'
```


还有一个稍微复杂一点的问题。


```python
chain.invoke({chain.input_key: "What is the climate on Luke Skywalker's home planet?"})[
    chain.output_key
]
```
```output


[1m> Entering new OntotextGraphDBQAChain chain...[0m
Generated SPARQL:
[32;1m[1;3mPREFIX : <https://swapi.co/vocabulary/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?climate
WHERE {
  ?character rdfs:label "Luke Skywalker" .
  ?character :homeworld ?planet .
  ?planet :climate ?climate .
}[0m

[1m> Finished chain.[0m
```


```output
"卢克·天行者的家乡星球的气候是干旱的。"
```


我们还可以问更复杂的问题，比如


```python
chain.invoke(
    {
        chain.input_key: "What is the average box office revenue for all the Star Wars movies?"
    }
)[chain.output_key]
```
```output


[1m> Entering new OntotextGraphDBQAChain chain...[0m
Generated SPARQL:
[32;1m[1;3mPREFIX : <https://swapi.co/vocabulary/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT (AVG(?boxOffice) AS ?averageBoxOffice)
WHERE {
  ?film a :Film .
  ?film :boxOffice ?boxOfficeValue .
  BIND(xsd:decimal(?boxOfficeValue) AS ?boxOffice)
}
[0m

[1m> Finished chain.[0m
```


```output
'所有星球大战电影的平均票房收入约为 7.541 亿美元。'
```

## 链修饰符

Ontotext GraphDB QA 链允许对提示进行优化，以进一步改善您的 QA 链并提升您应用程序的整体用户体验。

### "SPARQL 生成" 提示

该提示用于根据用户问题和知识图谱（KG）模式生成 SPARQL 查询。

- `sparql_generation_prompt`

    默认值：
  ````python
    GRAPHDB_SPARQL_GENERATION_TEMPLATE = """
    为查询图数据库编写 SPARQL SELECT 查询。
    以三重反引号分隔的本体模式（Turtle 格式）为：
    ```
    {schema}
    ```
    仅使用模式中提供的类和属性来构造 SPARQL 查询。
    不要使用在 SPARQL 查询中未明确提供的任何类或属性。
    包括所有必要的前缀。
    在您的响应中不要包含任何解释或道歉。
    不要将查询用反引号括起来。
    除生成的 SPARQL 查询外，不要包含任何文本。
    以三重反引号分隔的问题是：
    ```
    {prompt}
    ```
    """
    GRAPHDB_SPARQL_GENERATION_PROMPT = PromptTemplate(
        input_variables=["schema", "prompt"],
        template=GRAPHDB_SPARQL_GENERATION_TEMPLATE,
    )
  ````

### "SPARQL 修复" 提示

有时，LLM 可能会生成带有语法错误或缺少前缀等的 SPARQL 查询。该链将尝试通过提示 LLM 修正该查询若干次。

- `sparql_fix_prompt`

    默认值：
  ````python
    GRAPHDB_SPARQL_FIX_TEMPLATE = """
    以下 SPARQL 查询由三重反引号分隔
    ```
    {generated_sparql}
    ```
    是无效的。
    由三重反引号分隔的错误是
    ```
    {error_message}
    ```
    请给我一个正确版本的 SPARQL 查询。
    不要改变查询的逻辑。
    在您的回答中不要包含任何解释或道歉。
    不要将查询用反引号包裹。
    不要包含除生成的 SPARQL 查询外的任何文本。
    由三重反引号分隔的本体模式为 Turtle 格式：
    ```
    {schema}
    ```
    """
    
    GRAPHDB_SPARQL_FIX_PROMPT = PromptTemplate(
        input_variables=["error_message", "generated_sparql", "schema"],
        template=GRAPHDB_SPARQL_FIX_TEMPLATE,
    )
  ````

- `max_fix_retries`
  
    默认值： `5`

### "回答"提示

该提示用于根据从数据库返回的结果和初始用户问题回答问题。默认情况下，LLM 被指示仅使用返回结果中的信息。如果结果集为空，LLM 应告知无法回答该问题。

- `qa_prompt`

  默认值：
  ````python
    GRAPHDB_QA_TEMPLATE = """任务：根据 SPARQL 查询的结果生成自然语言响应。
    你是一个能够创建写得很好且易于理解的答案的助手。
    信息部分包含提供的信息，你可以使用这些信息来构建答案。
    提供的信息是权威的，你绝不能怀疑它或试图用你内部的知识来纠正它。
    让你的回答听起来像是来自 AI 助手的信息，但不要添加任何信息。
    如果没有可用的信息，不要使用内部知识来回答问题，只需说你不知道。
    信息：
    {context}
    
    问题：{prompt}
    有帮助的答案："""
    GRAPHDB_QA_PROMPT = PromptTemplate(
        input_variables=["context", "prompt"], template=GRAPHDB_QA_TEMPLATE
    )
  ````

完成与 GraphDB 的 QA 交互后，可以通过在包含 Docker compose 文件的目录中运行
``
docker compose down -v --remove-orphans
``
来关闭 Docker 环境。