---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/requests.ipynb
---

# 请求

网络上包含大量 LLM 无法访问的信息。为了方便 LLM 与这些信息进行交互，我们提供了一个封装 Python Requests 模块的工具，它接受一个 URL 并从该 URL 获取数据。

```python
%pip install --upgrade --quiet langchain-community
```

```python
from langchain_community.agent_toolkits.load_tools import load_tools

requests_tools = load_tools(["requests_all"], allow_dangerous_tools=True)

requests_tools
```

```output
[RequestsGetTool(requests_wrapper=TextRequestsWrapper(headers=None, aiosession=None, auth=None, response_content_type='text', verify=True), allow_dangerous_requests=True),
 RequestsPostTool(requests_wrapper=TextRequestsWrapper(headers=None, aiosession=None, auth=None, response_content_type='text', verify=True), allow_dangerous_requests=True),
 RequestsPatchTool(requests_wrapper=TextRequestsWrapper(headers=None, aiosession=None, auth=None, response_content_type='text', verify=True), allow_dangerous_requests=True),
 RequestsPutTool(requests_wrapper=TextRequestsWrapper(headers=None, aiosession=None, auth=None, response_content_type='text', verify=True), allow_dangerous_requests=True),
 RequestsDeleteTool(requests_wrapper=TextRequestsWrapper(headers=None, aiosession=None, auth=None, response_content_type='text', verify=True), allow_dangerous_requests=True)]
```

### 工具内部

每个请求工具包含一个 `requests` 包装器。您可以直接在下面使用这些包装器


```python
# 每个工具包装一个 requests 包装器
requests_tools[0].requests_wrapper
```



```output
TextRequestsWrapper(headers=None, aiosession=None, auth=None, response_content_type='text', verify=True)
```



```python
from langchain_community.utilities import TextRequestsWrapper

requests = TextRequestsWrapper()

requests.get("https://www.google.com")
```




如果您需要将输出解码为 JSON，您可以使用 ``JsonRequestsWrapper``。


```python
from langchain_community.utilities.requests import JsonRequestsWrapper

requests = JsonRequestsWrapper()


rval = requests.get("https://api.agify.io/?name=jackson")

print(
    f"""

类型 - {type(rval)}

内容: 
```
{rval}
```

"""
)
```
```output


类型 - <class 'dict'>

内容: 
```
{'count': 5707, 'name': 'jackson', 'age': 38}
```
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)