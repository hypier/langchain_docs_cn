---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/graphs/falkordb.ipynb
---

# FalkorDB

>[FalkorDB](https://www.falkordb.com/) 是一个低延迟的图数据库，为 GenAI 提供知识。

本笔记本展示了如何使用 LLMs 为 `FalkorDB` 数据库提供自然语言接口。

## 设置

您可以在本地运行 `falkordb` Docker 容器：

```bash
docker run -p 6379:6379 -it --rm falkordb/falkordb
```

启动后，您可以在本地机器上创建一个数据库并连接到它。

```python
from langchain.chains import FalkorDBQAChain
from langchain_community.graphs import FalkorDBGraph
from langchain_openai import ChatOpenAI
```

## 创建图连接并插入示例数据


```python
graph = FalkorDBGraph(database="movies")
```


```python
graph.query(
    """
    CREATE 
        (al:Person {name: 'Al Pacino', birthDate: '1940-04-25'}),
        (robert:Person {name: 'Robert De Niro', birthDate: '1943-08-17'}),
        (tom:Person {name: 'Tom Cruise', birthDate: '1962-07-3'}),
        (val:Person {name: 'Val Kilmer', birthDate: '1959-12-31'}),
        (anthony:Person {name: 'Anthony Edwards', birthDate: '1962-7-19'}),
        (meg:Person {name: 'Meg Ryan', birthDate: '1961-11-19'}),

        (god1:Movie {title: 'The Godfather'}),
        (god2:Movie {title: 'The Godfather: Part II'}),
        (god3:Movie {title: 'The Godfather Coda: The Death of Michael Corleone'}),
        (top:Movie {title: 'Top Gun'}),

        (al)-[:ACTED_IN]->(god1),
        (al)-[:ACTED_IN]->(god2),
        (al)-[:ACTED_IN]->(god3),
        (robert)-[:ACTED_IN]->(god2),
        (tom)-[:ACTED_IN]->(top),
        (val)-[:ACTED_IN]->(top),
        (anthony)-[:ACTED_IN]->(top),
        (meg)-[:ACTED_IN]->(top)
"""
)
```



```output
[]
```

## 创建 FalkorDBQAChain


```python
graph.refresh_schema()
print(graph.schema)

import os

os.environ["OPENAI_API_KEY"] = "API_KEY_HERE"
```
```output
节点属性: [[OrderedDict([('label', None), ('properties', ['name', 'birthDate', 'title'])])]]
关系属性: [[OrderedDict([('type', None), ('properties', [])])]]
关系: [['(:Person)-[:ACTED_IN]->(:Movie)']]
```

```python
chain = FalkorDBQAChain.from_llm(ChatOpenAI(temperature=0), graph=graph, verbose=True)
```

## 查询图形


```python
chain.run("Who played in Top Gun?")
```
```output


[1m> Entering new FalkorDBQAChain chain...[0m
Generated Cypher:
[32;1m[1;3mMATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE m.title = 'Top Gun'
RETURN p.name[0m
Full Context:
[32;1m[1;3m[['Tom Cruise'], ['Val Kilmer'], ['Anthony Edwards'], ['Meg Ryan'], ['Tom Cruise'], ['Val Kilmer'], ['Anthony Edwards'], ['Meg Ryan']][0m

[1m> Finished chain.[0m
```


```output
'汤姆·克鲁斯、瓦尔·基尔默、安东尼·爱德华兹和梅格·瑞恩在《壮志凌云》中出演。'
```



```python
chain.run("Who is the oldest actor who played in The Godfather: Part II?")
```
```output


[1m> Entering new FalkorDBQAChain chain...[0m
Generated Cypher:
[32;1m[1;3mMATCH (p:Person)-[r:ACTED_IN]->(m:Movie)
WHERE m.title = 'The Godfather: Part II'
RETURN p.name
ORDER BY p.birthDate ASC
LIMIT 1[0m
Full Context:
[32;1m[1;3m[['Al Pacino']][0m

[1m> Finished chain.[0m
```


```output
'在《教父：第二部》中出演的最年长演员是阿尔·帕西诺。'
```



```python
chain.run("Robert De Niro played in which movies?")
```
```output


[1m> Entering new FalkorDBQAChain chain...[0m
Generated Cypher:
[32;1m[1;3mMATCH (p:Person {name: 'Robert De Niro'})-[:ACTED_IN]->(m:Movie)
RETURN m.title[0m
Full Context:
[32;1m[1;3m[['The Godfather: Part II'], ['The Godfather: Part II']][0m

[1m> Finished chain.[0m
```


```output
'罗伯特·德尼罗出演了《教父：第二部》。'
```