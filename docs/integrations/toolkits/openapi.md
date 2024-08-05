---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/openapi.ipynb
---

# OpenAPI

我们可以构建代理来消费任意 API，这里是符合 `OpenAPI`/`Swagger` 规范的 API。

```python
# NOTE: In this example. We must set `allow_dangerous_request=True` to enable the OpenAPI Agent to automatically use the Request Tool.
# This can be dangerous for calling unwanted requests. Please make sure your custom OpenAPI spec (yaml) is safe.
ALLOW_DANGEROUS_REQUEST = True
```

## 1st example: hierarchical planning agent

在这个例子中，我们将考虑一种称为层次规划的方法，这在机器人技术中很常见，并出现在最近的 LLMs 与机器人技术的研究中。我们将看到这是一种可行的方法，可以开始处理庞大的 API 规范，并协助处理需要多个步骤的用户查询。

这个想法很简单：为了在长序列中获得一致的代理行为并节省令牌，我们将分离关注点：“规划者”将负责调用哪些端点，而“控制器”将负责如何调用它们。

在初始实现中，规划者是一个 LLM 链，其中包含每个端点的名称和简短描述。控制器是一个 LLM 代理，仅使用特定计划的端点文档进行实例化。还有很多工作要做，以使其非常稳健 :)

---

### 首先，让我们收集一些 OpenAPI 规范。


```python
import os

import yaml
```

您可以从这里获取 OpenAPI 规范：[APIs-guru/openapi-directory](https://github.com/APIs-guru/openapi-directory)


```python
!wget https://raw.githubusercontent.com/openai/openai-openapi/master/openapi.yaml -O openai_openapi.yaml
!wget https://www.klarna.com/us/shopping/public/openai/v0/api-docs -O klarna_openapi.yaml
!wget https://raw.githubusercontent.com/APIs-guru/openapi-directory/main/APIs/spotify.com/1.0.0/openapi.yaml -O spotify_openapi.yaml
```

```python
from langchain_community.agent_toolkits.openapi.spec import reduce_openapi_spec
```


```python
with open("openai_openapi.yaml") as f:
    raw_openai_api_spec = yaml.load(f, Loader=yaml.Loader)
openai_api_spec = reduce_openapi_spec(raw_openai_api_spec)

with open("klarna_openapi.yaml") as f:
    raw_klarna_api_spec = yaml.load(f, Loader=yaml.Loader)
klarna_api_spec = reduce_openapi_spec(raw_klarna_api_spec)

with open("spotify_openapi.yaml") as f:
    raw_spotify_api_spec = yaml.load(f, Loader=yaml.Loader)
spotify_api_spec = reduce_openapi_spec(raw_spotify_api_spec)
```

---

我们将使用 Spotify API 作为一个相对复杂的 API 的示例。如果您想要复制这个过程，需要进行一些与身份验证相关的设置。

- 您需要在 Spotify 开发者控制台中设置一个应用程序，文档见 [这里](https://developer.spotify.com/documentation/general/guides/authorization/)，以获取凭据：`CLIENT_ID`、`CLIENT_SECRET` 和 `REDIRECT_URI`。
- 要获取访问令牌（并保持其有效），您可以实现 oauth 流程，或者使用 `spotipy`。如果您已将 Spotify 凭据设置为环境变量 `SPOTIPY_CLIENT_ID`、`SPOTIPY_CLIENT_SECRET` 和 `SPOTIPY_REDIRECT_URI`，您可以使用以下辅助函数：


```python
import spotipy.util as util
from langchain.requests import RequestsWrapper


def construct_spotify_auth_headers(raw_spec: dict):
    scopes = list(
        raw_spec["components"]["securitySchemes"]["oauth_2_0"]["flows"][
            "authorizationCode"
        ]["scopes"].keys()
    )
    access_token = util.prompt_for_user_token(scope=",".join(scopes))
    return {"Authorization": f"Bearer {access_token}"}


# 获取 API 凭据。
headers = construct_spotify_auth_headers(raw_spotify_api_spec)
requests_wrapper = RequestsWrapper(headers=headers)
```

### 这个规格有多大？


```python
endpoints = [
    (route, operation)
    for route, operations in raw_spotify_api_spec["paths"].items()
    for operation in operations
    if operation in ["get", "post"]
]
len(endpoints)
```



```output
63
```



```python
import tiktoken

enc = tiktoken.encoding_for_model("gpt-4")


def count_tokens(s):
    return len(enc.encode(s))


count_tokens(yaml.dump(raw_spotify_api_spec))
```



```output
80326
```

### 让我们看一些例子！

从 GPT-4 开始。（GPT-3 系列正在进行一些稳健性迭代。）

```python
from langchain_community.agent_toolkits.openapi import planner
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model_name="gpt-4", temperature=0.0)
```
```output
/Users/jeremywelborn/src/langchain/langchain/llms/openai.py:169: UserWarning: You are trying to use a chat model. This way of initializing it is no longer supported. Instead, please use: `from langchain_openai import ChatOpenAI`
  warnings.warn(
/Users/jeremywelborn/src/langchain/langchain/llms/openai.py:608: UserWarning: You are trying to use a chat model. This way of initializing it is no longer supported. Instead, please use: `from langchain_openai import ChatOpenAI`
  warnings.warn(
```

```python
# 注意：出于安全考虑，请手动设置 allow_dangerous_requests https://python.langchain.com/docs/security
spotify_agent = planner.create_openapi_agent(
    spotify_api_spec,
    requests_wrapper,
    llm,
    allow_dangerous_requests=ALLOW_DANGEROUS_REQUEST,
)
user_query = (
    "为我制作一个包含《蓝调之种》第一首歌的播放列表。叫它机器蓝调。"
)
spotify_agent.invoke(user_query)
```



```output
'我创建了一个名为 "Machine Blues" 的播放列表，其中包含《Kind of Blue》专辑的第一首歌。'
```



```python
user_query = "给我推荐一首我会喜欢的歌，最好是蓝调风格"
spotify_agent.invoke(user_query)
```



```output
'推荐给你的蓝调歌曲是 "Get Away Jordan"，曲目 ID: 03lXHmokj9qsXspNsPoirR。'
```


#### 尝试另一个 API。



```python
headers = {"Authorization": f"Bearer {os.getenv('OPENAI_API_KEY')}"}
openai_requests_wrapper = RequestsWrapper(headers=headers)
```


```python
# Meta!
llm = ChatOpenAI(model_name="gpt-4", temperature=0.25)
openai_agent = planner.create_openapi_agent(
    openai_api_spec, openai_requests_wrapper, llm
)
user_query = "生成一条简短的建议"
openai_agent.invoke(user_query)
```



```output
'改善沟通技巧的简短建议是确保倾听。'
```


需要一些时间才能达到目标！

## 第二个示例：“json explorer”代理

这是一个不太实用但很有趣的代理！该代理可以访问两个工具包。一个工具包包含与json交互的工具：一个工具用于列出json对象的键，另一个工具用于获取给定键的值。另一个工具包包含发送GET和POST请求的`requests`包装器。这个代理消耗了大量对语言模型的调用，但表现得相当不错。



```python
from langchain_community.agent_toolkits import OpenAPIToolkit, create_openapi_agent
from langchain_community.tools.json.tool import JsonSpec
from langchain_openai import OpenAI
```


```python
with open("openai_openapi.yaml") as f:
    data = yaml.load(f, Loader=yaml.FullLoader)
json_spec = JsonSpec(dict_=data, max_value_length=4000)


openapi_toolkit = OpenAPIToolkit.from_llm(
    OpenAI(temperature=0), json_spec, openai_requests_wrapper, verbose=True
)
openapi_agent_executor = create_openapi_agent(
    llm=OpenAI(temperature=0),
    toolkit=openapi_toolkit,
    allow_dangerous_requests=ALLOW_DANGEROUS_REQUEST,
    verbose=True,
)
```


```python
openapi_agent_executor.run(
    "Make a post request to openai /completions. The prompt should be 'tell me a joke.'"
)
```