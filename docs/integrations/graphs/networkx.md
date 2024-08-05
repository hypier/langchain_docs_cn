---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/graphs/networkx.ipynb
---

# NetworkX

>[NetworkX](https://networkx.org/) 是一个用于创建、操作和研究复杂网络的结构、动态和功能的 Python 包。

本笔记本介绍如何在图数据结构上进行问答。

## 设置

我们需要安装一个 Python 包。


```python
%pip install --upgrade --quiet  networkx
```

## 创建图形

在本节中，我们构建一个示例图形。目前，这对于小段文本效果最佳。


```python
from langchain.indexes import GraphIndexCreator
from langchain_openai import OpenAI
```


```python
index_creator = GraphIndexCreator(llm=OpenAI(temperature=0))
```


```python
with open("../../../how_to/state_of_the_union.txt") as f:
    all_text = f.read()
```

我们只会使用一个小片段，因为提取知识三元组目前有点费时。


```python
text = "\n".join(all_text.split("\n\n")[105:108])
```


```python
text
```



```output
'It won’t look like much, but if you stop and look closely, you’ll see a “Field of dreams,” the ground on which America’s future will be built. \nThis is where Intel, the American company that helped build Silicon Valley, is going to build its $20 billion semiconductor “mega site”. \nUp to eight state-of-the-art factories in one place. 10,000 new good-paying jobs. '
```



```python
graph = index_creator.from_text(text)
```

我们可以查看创建的图形。


```python
graph.get_triples()
```



```output
[('Intel', '$20 billion semiconductor "mega site"', 'is going to build'),
 ('Intel', 'state-of-the-art factories', 'is building'),
 ('Intel', '10,000 new good-paying jobs', 'is creating'),
 ('Intel', 'Silicon Valley', 'is helping build'),
 ('Field of dreams',
  "America's future will be built",
  'is the ground on which')]
```

## 查询图形
我们现在可以使用图形 QA 链来询问图形

```python
from langchain.chains import GraphQAChain
```

```python
chain = GraphQAChain.from_llm(OpenAI(temperature=0), graph=graph, verbose=True)
```

```python
chain.run("Intel 将要建设什么？")
```
```output


[1m> 进入新的 GraphQAChain 链...[0m
提取的实体：
[32;1m[1;3m Intel[0m
完整上下文：
[32;1m[1;3mIntel 将要建设价值 200 亿美元的半导体“超级工厂”
Intel 正在建设最先进的工厂
Intel 正在创造 10,000 个新的高薪工作
Intel 正在帮助建设硅谷[0m

[1m> 完成链。[0m
```

```output
' Intel 将要建设一个价值 200 亿美元的半导体“超级工厂”，配备最先进的工厂，创造 10,000 个新的高薪工作，并帮助建设硅谷。'
```

## 保存图形
我们还可以保存和加载图形。

```python
graph.write_to_gml("graph.gml")
```

```python
from langchain.indexes.graph import NetworkxEntityGraph
```

```python
loaded_graph = NetworkxEntityGraph.from_gml("graph.gml")
```

```python
loaded_graph.get_triples()
```

```output
[('Intel', '$20 billion semiconductor "mega site"', 'is going to build'),
 ('Intel', 'state-of-the-art factories', 'is building'),
 ('Intel', '10,000 new good-paying jobs', 'is creating'),
 ('Intel', 'Silicon Valley', 'is helping build'),
 ('Field of dreams',
  "America's future will be built",
  'is the ground on which')]
```

```python
loaded_graph.get_number_of_nodes()
```

```python
loaded_graph.add_node("NewNode")
```

```python
loaded_graph.has_node("NewNode")
```

```python
loaded_graph.remove_node("NewNode")
```

```python
loaded_graph.get_neighbors("Intel")
```

```python
loaded_graph.has_edge("Intel", "Silicon Valley")
```

```python
loaded_graph.remove_edge("Intel", "Silicon Valley")
```

```python
loaded_graph.clear_edges()
```

```python
loaded_graph.clear()
```