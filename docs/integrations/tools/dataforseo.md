---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/dataforseo.ipynb
---

# DataForSEO

>[DataForSeo](https://dataforseo.com/) 提供全面的 SEO 和数字营销数据解决方案，通过 API 实现。
>
>`DataForSeo API` 从最流行的搜索引擎如 `Google`、`Bing`、`Yahoo` 获取 `SERP`。它还允许从不同类型的搜索引擎获取 SERP，如 `Maps`、`News`、`Events` 等。

本笔记本演示如何使用 [DataForSeo API](https://dataforseo.com/apis) 获取搜索引擎结果。 


```python
%pip install --upgrade --quiet langchain-community
```


```python
from langchain_community.utilities.dataforseo_api_search import DataForSeoAPIWrapper
```

## 设置 API 凭证

您可以通过在 `DataForSeo` 网站上注册来获取您的 API 凭证。


```python
import os

os.environ["DATAFORSEO_LOGIN"] = "your_api_access_username"
os.environ["DATAFORSEO_PASSWORD"] = "your_api_access_password"

wrapper = DataForSeoAPIWrapper()
```

run 方法将返回以下元素之一的第一个结果片段：answer_box、knowledge_graph、featured_snippet、shopping、organic。


```python
wrapper.run("Weather in Los Angeles")
```

## `run` 和 `results` 的区别
`run` 和 `results` 是 `DataForSeoAPIWrapper` 类提供的两种方法。

`run` 方法执行搜索并返回来自答案框、知识图谱、精选片段、购物或自然结果的第一个结果片段。这些元素按优先级从高到低排序。

`results` 方法返回根据包装器中设置的参数配置的 JSON 响应。这使得您在希望从 API 返回哪些数据时具有更大的灵活性。

## 以 JSON 格式获取结果
您可以自定义希望在 JSON 响应中返回的结果类型和字段。您还可以设置要返回的顶部结果的最大数量。

```python
json_wrapper = DataForSeoAPIWrapper(
    json_result_types=["organic", "knowledge_graph", "answer_box"],
    json_result_fields=["type", "title", "description", "text"],
    top_count=3,
)
```

```python
json_wrapper.results("Bill Gates")
```

## 自定义位置和语言
您可以通过向 API 包装器传递附加参数来指定搜索结果的位置和语言。

```python
customized_wrapper = DataForSeoAPIWrapper(
    top_count=10,
    json_result_types=["organic", "local_pack"],
    json_result_fields=["title", "description", "type"],
    params={"location_name": "Germany", "language_code": "en"},
)
customized_wrapper.results("coffee near me")
```

## 自定义搜索引擎
您还可以指定要使用的搜索引擎。

```python
customized_wrapper = DataForSeoAPIWrapper(
    top_count=10,
    json_result_types=["organic", "local_pack"],
    json_result_fields=["title", "description", "type"],
    params={"location_name": "Germany", "language_code": "en", "se_name": "bing"},
)
customized_wrapper.results("coffee near me")
```

## 自定义搜索类型
API 封装还允许您指定要执行的搜索类型。例如，您可以执行地图搜索。

```python
maps_search = DataForSeoAPIWrapper(
    top_count=10,
    json_result_fields=["title", "value", "address", "rating", "type"],
    params={
        "location_coordinate": "52.512,13.36,12z",
        "language_code": "en",
        "se_type": "maps",
    },
)
maps_search.results("coffee near me")
```

## 与 Langchain 代理的集成
您可以使用 `langchain.agents` 模块中的 `Tool` 类将 `DataForSeoAPIWrapper` 集成到 langchain 代理中。`Tool` 类封装了代理可以调用的函数。

```python
from langchain_core.tools import Tool

search = DataForSeoAPIWrapper(
    top_count=3,
    json_result_types=["organic"],
    json_result_fields=["title", "description", "type"],
)
tool = Tool(
    name="google-search-answer",
    description="My new answer tool",
    func=search.run,
)
json_tool = Tool(
    name="google-search-json",
    description="My new json tool",
    func=search.results,
)
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)