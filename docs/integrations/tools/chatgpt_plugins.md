---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/chatgpt_plugins.ipynb
sidebar_class_name: hidden
---

# ChatGPT 插件

:::warning 已弃用

OpenAI 已经 [弃用插件](https://openai.com/index/chatgpt-plugins/)。

:::

此示例展示了如何在 LangChain 抽象中使用 ChatGPT 插件。

注意 1：目前仅适用于没有身份验证的插件。

注意 2：几乎肯定还有其他方法可以做到这一点，这只是初步的尝试。如果您有更好的想法，请提交 PR！


```python
%pip install --upgrade --quiet langchain-community
```


```python
from langchain_community.tools import AIPluginTool
```


```python
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import ChatOpenAI
```


```python
tool = AIPluginTool.from_plugin_url("https://www.klarna.com/.well-known/ai-plugin.json")
```


```python
llm = ChatOpenAI(temperature=0)
tools = load_tools(["requests_all"])
tools += [tool]

agent_chain = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
agent_chain.run("what t shirts are available in klarna?")
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m我需要检查 Klarna 购物 API 以查看是否有关于可用 T 恤的信息。
操作：KlarnaProducts
操作输入：无[0m
观察：[33;1m[1;3m使用指南：使用 Klarna 插件获取任何购物或研究目的的相关产品建议。发送的查询不应包含像冠词、介词和限定词这样的停用词。API 在搜索与产品相关的词时效果最佳，例如它们的名称、品牌、型号或类别。链接将始终返回，并应显示给用户。

OpenAPI 规范：{'openapi': '3.0.1', 'info': {'version': 'v0', 'title': 'Open AI Klarna product Api'}, 'servers': [{'url': 'https://www.klarna.com/us/shopping'}], 'tags': [{'name': 'open-ai-product-endpoint', 'description': 'Open AI Product Endpoint. Query for products.'}], 'paths': {'/public/openai/v0/products': {'get': {'tags': ['open-ai-product-endpoint'], 'summary': '用于获取 Klarna 产品信息的 API', 'operationId': 'productsUsingGET', 'parameters': [{'name': 'q', 'in': 'query', 'description': '查询，必须在 2 到 100 个字符之间', 'required': True, 'schema': {'type': 'string'}}, {'name': 'size', 'in': 'query', 'description': '返回的产品数量', 'required': False, 'schema': {'type': 'integer'}}, {'name': 'budget', 'in': 'query', 'description': '匹配产品的最高价格（以当地货币），过滤结果', 'required': False, 'schema': {'type': 'integer'}}], 'responses': {'200': {'description': '找到的产品', 'content': {'application/json': {'schema': {'$ref': '#/components/schemas/ProductResponse'}}}}, '503': {'description': '一个或多个服务不可用'}}, 'deprecated': False}}}, 'components': {'schemas': {'Product': {'type': 'object', 'properties': {'attributes': {'type': 'array', 'items': {'type': 'string'}}, 'name': {'type': 'string'}, 'price': {'type': 'string'}, 'url': {'type': 'string'}}, 'title': 'Product'}, 'ProductResponse': {'type': 'object', 'properties': {'products': {'type': 'array', 'items': {'$ref': '#/components/schemas/Product'}}}, 'title': 'ProductResponse'}}}}[0m
思考：[32;1m[1;3m我需要使用 Klarna 购物 API 搜索 T 恤。
操作：requests_get
操作输入：https://www.klarna.com/us/shopping/public/openai/v0/products?q=t%20shirts[0m
观察：[36;1m[1;3m{"products":[{"name":"Lacoste Men's Pack of Plain T-Shirts","url":"https://www.klarna.com/us/shopping/pl/cl10001/3202043025/Clothing/Lacoste-Men-s-Pack-of-Plain-T-Shirts/?utm_source=openai","price":"$26.60","attributes":["Material:Cotton","Target Group:Man","Color:White,Black"]},{"name":"Hanes Men's Ultimate 6pk. Crewneck T-Shirts","url":"https://www.klarna.com/us/shopping/pl/cl10001/3201808270/Clothing/Hanes-Men-s-Ultimate-6pk.-Crewneck-T-Shirts/?utm_source=openai","price":"$13.82","attributes":["Material:Cotton","Target Group:Man","Color:White"]},{"name":"Nike Boy's Jordan Stretch T-shirts","url":"https://www.klarna.com/us/shopping/pl/cl359/3201863202/Children-s-Clothing/Nike-Boy-s-Jordan-Stretch-T-shirts/?utm_source=openai","price":"$14.99","attributes":["Material:Cotton","Color:White,Green","Model:Boy","Size (Small-Large):S,XL,L,M"]},{"name":"Polo Classic Fit Cotton V-Neck T-Shirts 3-Pack","url":"https://www.klarna.com/us/shopping/pl/cl10001/3203028500/Clothing/Polo-Classic-Fit-Cotton-V-Neck-T-Shirts-3-Pack/?utm_source=openai","price":"$29.95","attributes":["Material:Cotton","Target Group:Man","Color:White,Blue,Black"]},{"name":"adidas Comfort T-shirts Men's 3-pack","url":"https://www.klarna.com/us/shopping/pl/cl10001/3202640533/Clothing/adidas-Comfort-T-shirts-Men-s-3-pack/?utm_source=openai","price":"$14.99","attributes":["Material:Cotton","Target Group:Man","Color:White,Black","Neckline:Round"]}]}[0m
思考：[32;1m[1;3mKlarna 中可用的 T 恤包括 Lacoste 男士纯色 T 恤、Hanes 男士终极 6 件套圆领 T 恤、Nike 男孩乔丹弹力 T 恤、Polo 经典款棉质 V 领 T 恤 3 件套，以及 adidas 舒适 T 恤男士 3 件套。
最终答案：Klarna 中可用的 T 恤包括 Lacoste 男士纯色 T 恤、Hanes 男士终极 6 件套圆领 T 恤、Nike 男孩乔丹弹力 T 恤、Polo 经典款棉质 V 领 T 恤 3 件套，以及 adidas 舒适 T 恤男士 3 件套。[0m

[1m> 完成链。[0m
```


```output
"Klarna 中可用的 T 恤包括 Lacoste 男士纯色 T 恤、Hanes 男士终极 6 件套圆领 T 恤、Nike 男孩乔丹弹力 T 恤、Polo 经典款棉质 V 领 T 恤 3 件套，以及 adidas 舒适 T 恤男士 3 件套。"
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)