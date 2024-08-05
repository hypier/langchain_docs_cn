---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/graphs/nebula_graph.ipynb
---

# NebulaGraph

>[NebulaGraph](https://www.nebula-graph.io/) 是一个开源的、分布式的、可扩展的、快速的图数据库，专为超大规模图形构建，延迟为毫秒级。它使用 `nGQL` 图查询语言。
>
>[nGQL](https://docs.nebula-graph.io/3.0.0/3.ngql-guide/1.nGQL-overview/1.overview/) 是用于 `NebulaGraph` 的声明式图查询语言。它允许表达丰富且高效的图模式。`nGQL` 旨在服务于开发人员和运维专业人员。`nGQL` 是一种类似 SQL 的查询语言。

本笔记本展示了如何使用 LLM 提供自然语言接口来访问 `NebulaGraph` 数据库。

## 设置

您可以通过运行以下脚本以 Docker 容器的形式启动 `NebulaGraph` 集群：

```bash
curl -fsSL nebula-up.siwei.io/install.sh | bash
```

其他选项包括：
- 作为 [Docker Desktop 扩展](https://www.docker.com/blog/distributed-cloud-native-graph-database-nebulagraph-docker-extension/) 安装。请参见 [这里](https://docs.nebula-graph.io/3.5.0/2.quick-start/1.quick-start-workflow/)
- NebulaGraph 云服务。请参见 [这里](https://www.nebula-graph.io/cloud)
- 从包、源代码或通过 Kubernetes 部署。请参见 [这里](https://docs.nebula-graph.io/)

一旦集群运行，我们可以为数据库创建 `SPACE` 和 `SCHEMA`。

```python
%pip install --upgrade --quiet  ipython-ngql
%load_ext ngql

# 连接 ngql jupyter 扩展到 nebulagraph
%ngql --address 127.0.0.1 --port 9669 --user root --password nebula
# 创建一个新空间
%ngql CREATE SPACE IF NOT EXISTS langchain(partition_num=1, replica_factor=1, vid_type=fixed_string(128));
```

```python
# 等待几秒钟以创建空间。
%ngql USE langchain;
```

创建模式，完整数据集请参考 [这里](https://www.siwei.io/en/nebulagraph-etl-dbt/)。

```python
%%ngql
CREATE TAG IF NOT EXISTS movie(name string);
CREATE TAG IF NOT EXISTS person(name string, birthdate string);
CREATE EDGE IF NOT EXISTS acted_in();
CREATE TAG INDEX IF NOT EXISTS person_index ON person(name(128));
CREATE TAG INDEX IF NOT EXISTS movie_index ON movie(name(128));
```

等待模式创建完成，然后我们可以插入一些数据。

```python
%%ngql
INSERT VERTEX person(name, birthdate) VALUES "Al Pacino":("Al Pacino", "1940-04-25");
INSERT VERTEX movie(name) VALUES "The Godfather II":("The Godfather II");
INSERT VERTEX movie(name) VALUES "The Godfather Coda: The Death of Michael Corleone":("The Godfather Coda: The Death of Michael Corleone");
INSERT EDGE acted_in() VALUES "Al Pacino"->"The Godfather II":();
INSERT EDGE acted_in() VALUES "Al Pacino"->"The Godfather Coda: The Death of Michael Corleone":();
```

```python
from langchain.chains import NebulaGraphQAChain
from langchain_community.graphs import NebulaGraph
from langchain_openai import ChatOpenAI
```

```python
graph = NebulaGraph(
    space="langchain",
    username="root",
    password="nebula",
    address="127.0.0.1",
    port=9669,
    session_pool_size=30,
)
```

## 刷新图形模式信息

如果数据库的模式发生变化，您可以刷新生成 nGQL 语句所需的模式信息。

```python
# graph.refresh_schema()
```

```python
print(graph.get_schema)
```
```output
Node properties: [{'tag': 'movie', 'properties': [('name', 'string')]}, {'tag': 'person', 'properties': [('name', 'string'), ('birthdate', 'string')]}]
Edge properties: [{'edge': 'acted_in', 'properties': []}]
Relationships: ['(:person)-[:acted_in]->(:movie)']
```

## 查询图形

我们现在可以使用图形 cypher QA 链来询问图形

```python
chain = NebulaGraphQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True
)
```

```python
chain.run("Who played in The Godfather II?")
```
```output


[1m> 进入新的 NebulaGraphQAChain 链...[0m
生成的 nGQL:
[32;1m[1;3mMATCH (p:`person`)-[:acted_in]->(m:`movie`) WHERE m.`movie`.`name` == 'The Godfather II'
RETURN p.`person`.`name`[0m
完整上下文:
[32;1m[1;3m{'p.person.name': ['Al Pacino']}[0m

[1m> 完成链。[0m
```

```output
'Al Pacino played in The Godfather II.'
```