---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/openapi_nla.ipynb
---

# 自然语言API

`Natural Language API`工具包(`NLAToolkits`)允许LangChain代理有效地规划和组合端点之间的调用。

本笔记本演示了`Speak`、`Klarna`和`Spoonacluar` API的示例组合。

### 首先，导入依赖项并加载 LLM


```python
from langchain.agents import AgentType, initialize_agent
from langchain_community.agent_toolkits import NLAToolkit
from langchain_community.utilities import Requests
from langchain_openai import OpenAI
```


```python
# 选择要使用的 LLM。在这里，我们使用 gpt-3.5-turbo-instruct
llm = OpenAI(
    temperature=0, max_tokens=700, model_name="gpt-3.5-turbo-instruct"
)  # 您可以在不同的核心 LLM 之间进行切换。
```

### 接下来，加载自然语言 API 工具包


```python
speak_toolkit = NLAToolkit.from_llm_and_url(llm, "https://api.speak.com/openapi.yaml")
klarna_toolkit = NLAToolkit.from_llm_and_url(
    llm, "https://www.klarna.com/us/shopping/public/openai/v0/api-docs/"
)
```
```output
尝试加载 OpenAPI 3.0.1 规范。这可能导致性能下降。请将您的 OpenAPI 规范转换为 3.1.* 规范以获得更好的支持。
尝试加载 OpenAPI 3.0.1 规范。这可能导致性能下降。请将您的 OpenAPI 规范转换为 3.1.* 规范以获得更好的支持。
尝试加载 OpenAPI 3.0.1 规范。这可能导致性能下降。请将您的 OpenAPI 规范转换为 3.1.* 规范以获得更好的支持。
```

### 创建代理


```python
# 稍微调整默认代理的指令
openapi_format_instructions = """使用以下格式：

问题：您必须回答的输入问题
思考：您应该始终考虑该做什么
行动：要采取的行动，应为[{tool_names}]之一
行动输入：指示AI行动代表的内容。
观察：代理的响应
...（此思考/行动/行动输入/观察可以重复N次）
思考：我现在知道最终答案。用户无法看到我的任何观察、API响应、链接或工具。
最终答案：对原始输入问题的最终答案，包含适当的细节

在回应您的最终答案时，请记住，您所回应的人无法看到您的任何思考/行动/行动输入/观察，因此如果有任何相关信息，您需要在响应中明确包含它。"""
```


```python
natural_language_tools = speak_toolkit.get_tools() + klarna_toolkit.get_tools()
mrkl = initialize_agent(
    natural_language_tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
    agent_kwargs={"format_instructions": openapi_format_instructions},
)
```


```python
mrkl.run(
    "我有一个意大利语班的年终派对，必须为此购买一些意大利服装"
)
```
```output


[1m> 进入新的AgentExecutor链...[0m
[32;1m[1;3m 我需要找出有哪些意大利服装可供选择
行动：Open_AI_Klarna_product_Api.productsUsingGET
行动输入：意大利服装[0m
观察：[31;1m[1;3mAPI响应包含两个来自Alé品牌的意大利蓝色产品。第一个是Alé Colour Block短袖男士运动衫 - 意大利蓝，价格为$86.49，第二个是Alé Dolid Flash男士运动衫 - 意大利蓝，价格为$40.00。[0m
思考：[32;1m[1;3m 我现在知道有哪些意大利服装可供选择以及它们的价格。
最终答案：您可以为年终派对购买两款来自Alé品牌的意大利蓝色产品。Alé Colour Block短袖男士运动衫 - 意大利蓝的价格为$86.49，Alé Dolid Flash男士运动衫 - 意大利蓝的价格为$40.00。[0m

[1m> 完成链。[0m
```


```output
'您可以为年终派对购买两款来自Alé品牌的意大利蓝色产品。Alé Colour Block短袖男士运动衫 - 意大利蓝的价格为$86.49，Alé Dolid Flash男士运动衫 - 意大利蓝的价格为$40.00。'
```

### 使用身份验证并添加更多端点

某些端点可能需要通过访问令牌等方式进行用户身份验证。在这里，我们展示如何通过 `Requests` 包装对象传递身份验证信息。

由于每个 NLATool 都向其包装的 API 提供了简明的自然语言接口，因此顶层对话代理在整合每个端点以满足用户请求时工作更轻松。

**添加 Spoonacular 端点。**

1. 前往 [Spoonacular API 控制台](https://spoonacular.com/food-api/console#Profile) 并注册一个免费账户。
2. 点击 `Profile` 并复制下面的 API 密钥。


```python
spoonacular_api_key = ""  # Copy from the API Console
```


```python
requests = Requests(headers={"x-api-key": spoonacular_api_key})
spoonacular_toolkit = NLAToolkit.from_llm_and_url(
    llm,
    "https://spoonacular.com/application/frontend/downloads/spoonacular-openapi-3.json",
    requests=requests,
    max_text_length=1800,  # If you want to truncate the response text
)
```


```python
natural_language_api_tools = (
    speak_toolkit.get_tools()
    + klarna_toolkit.get_tools()
    + spoonacular_toolkit.get_tools()[:30]
)
print(f"{len(natural_language_api_tools)} tools loaded.")
```
```output
34 tools loaded.
```

```python
# Create an agent with the new tools
mrkl = initialize_agent(
    natural_language_api_tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
    agent_kwargs={"format_instructions": openapi_format_instructions},
)
```


```python
# Make the query more complex!
user_input = (
    "I'm learning Italian, and my language class is having an end of year party... "
    " Could you help me find an Italian outfit to wear and"
    " an appropriate recipe to prepare so I can present for the class in Italian?"
)
```


```python
mrkl.run(user_input)
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m I need to find a recipe and an outfit that is Italian-themed.
Action: spoonacular_API.searchRecipes
Action Input: Italian[0m
Observation: [36;1m[1;3mThe API response contains 10 Italian recipes, including Turkey Tomato Cheese Pizza, Broccolini Quinoa Pilaf, Bruschetta Style Pork & Pasta, Salmon Quinoa Risotto, Italian Tuna Pasta, Roasted Brussels Sprouts With Garlic, Asparagus Lemon Risotto, Italian Steamed Artichokes, Crispy Italian Cauliflower Poppers Appetizer, and Pappa Al Pomodoro.[0m
Thought:[32;1m[1;3m I need to find an Italian-themed outfit.
Action: Open_AI_Klarna_product_Api.productsUsingGET
Action Input: Italian[0m
Observation: [31;1m[1;3mI found 10 products related to 'Italian' in the API response. These products include Italian Gold Sparkle Perfectina Necklace - Gold, Italian Design Miami Cuban Link Chain Necklace - Gold, Italian Gold Miami Cuban Link Chain Necklace - Gold, Italian Gold Herringbone Necklace - Gold, Italian Gold Claddagh Ring - Gold, Italian Gold Herringbone Chain Necklace - Gold, Garmin QuickFit 22mm Italian Vacchetta Leather Band, Macy's Italian Horn Charm - Gold, Dolce & Gabbana Light Blue Italian Love Pour Homme EdT 1.7 fl oz.[0m
Thought:[32;1m[1;3m I now know the final answer.
Final Answer: To present for your Italian language class, you could wear an Italian Gold Sparkle Perfectina Necklace - Gold, an Italian Design Miami Cuban Link Chain Necklace - Gold, or an Italian Gold Miami Cuban Link Chain Necklace - Gold. For a recipe, you could make Turkey Tomato Cheese Pizza, Broccolini Quinoa Pilaf, Bruschetta Style Pork & Pasta, Salmon Quinoa Risotto, Italian Tuna Pasta, Roasted Brussels Sprouts With Garlic, Asparagus Lemon Risotto, Italian Steamed Artichokes, Crispy Italian Cauliflower Poppers Appetizer, or Pappa Al Pomodoro.[0m

[1m> Finished chain.[0m
```


```output
'To present for your Italian language class, you could wear an Italian Gold Sparkle Perfectina Necklace - Gold, an Italian Design Miami Cuban Link Chain Necklace - Gold, or an Italian Gold Miami Cuban Link Chain Necklace - Gold. For a recipe, you could make Turkey Tomato Cheese Pizza, Broccolini Quinoa Pilaf, Bruschetta Style Pork & Pasta, Salmon Quinoa Risotto, Italian Tuna Pasta, Roasted Brussels Sprouts With Garlic, Asparagus Lemon Risotto, Italian Steamed Artichokes, Crispy Italian Cauliflower Poppers Appetizer, or Pappa Al Pomodoro.'
```

## 谢谢！


```python
natural_language_api_tools[1].run(
    "Tell the LangChain audience to 'enjoy the meal' in Italian, please!"
)
```



```output
"In Italian, you can say 'Buon appetito' to someone to wish them to enjoy their meal. This phrase is commonly used in Italy when someone is about to eat, often at the beginning of a meal. It's similar to saying 'Bon appétit' in French or 'Guten Appetit' in German."
```