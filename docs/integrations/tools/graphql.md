---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/graphql.ipynb
---

# GraphQL

>[GraphQL](https://graphql.org/) 是一种用于 API 的查询语言，以及执行这些查询以访问数据的运行时。 `GraphQL` 提供了 API 中数据的完整且易于理解的描述，使客户端能够精确请求所需的内容，而不是多余的内容，简化了 API 随时间的演变，并支持强大的开发者工具。

通过在提供给代理的工具列表中包含 `BaseGraphQLTool`，您可以赋予代理从 GraphQL API 查询数据的能力，以满足您所需的任何目的。

此 Jupyter Notebook 演示了如何使用 `GraphQLAPIWrapper` 组件与代理。

在此示例中，我们将使用可在以下端点访问的公共 `Star Wars GraphQL API`： https://swapi-graphql.netlify.app/.netlify/functions/index。

首先，您需要安装 `httpx` 和 `gql` Python 包。

```python
pip install httpx gql > /dev/null
```

```python
%pip install --upgrade --quiet  langchain-community
```

现在，让我们创建一个带有指定星球大战 API 端点的 BaseGraphQLTool 实例，并用该工具初始化一个代理。

```python
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import OpenAI

llm = OpenAI(temperature=0)

tools = load_tools(
    ["graphql"],
    graphql_endpoint="https://swapi-graphql.netlify.app/.netlify/functions/index",
)

agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```

现在，我们可以使用代理对星球大战 GraphQL API 运行查询。让我们请求代理列出所有星球大战电影及其上映日期。

```python
graphql_fields = """allFilms {
    films {
      title
      director
      releaseDate
      speciesConnection {
        species {
          name
          classification
          homeworld {
            name
          }
        }
      }
    }
  }

"""

suffix = "Search for the titles of all the stawars films stored in the graphql database that has this schema "

agent.run(suffix + graphql_fields)
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m I need to query the graphql database to get the titles of all the star wars films
Action: query_graphql
Action Input: query { allFilms { films { title } } }[0m
Observation: [36;1m[1;3m"{\n  \"allFilms\": {\n    \"films\": [\n      {\n        \"title\": \"A New Hope\"\n      },\n      {\n        \"title\": \"The Empire Strikes Back\"\n      },\n      {\n        \"title\": \"Return of the Jedi\"\n      },\n      {\n        \"title\": \"The Phantom Menace\"\n      },\n      {\n        \"title\": \"Attack of the Clones\"\n      },\n      {\n        \"title\": \"Revenge of the Sith\"\n      }\n    ]\n  }\n}"[0m
Thought:[32;1m[1;3m I now know the titles of all the star wars films
Final Answer: The titles of all the star wars films are: A New Hope, The Empire Strikes Back, Return of the Jedi, The Phantom Menace, Attack of the Clones, and Revenge of the Sith.[0m

[1m> Finished chain.[0m
```


```output
'The titles of all the star wars films are: A New Hope, The Empire Strikes Back, Return of the Jedi, The Phantom Menace, Attack of the Clones, and Revenge of the Sith.'
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)