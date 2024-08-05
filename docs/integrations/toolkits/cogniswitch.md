---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/cogniswitch.ipynb
---

## Cogniswitch 工具

**使用 CogniSwitch 构建可以无缝获取、组织和检索知识的生产就绪应用程序。使用您选择的框架，在本例中是 Langchain，CogniSwitch 有助于减轻在选择合适的存储和检索格式时的决策压力。它还消除了生成响应时的可靠性问题和幻觉。只需两个简单步骤即可开始与您的知识互动。**

访问 [https://www.cogniswitch.ai/developer 注册](https://www.cogniswitch.ai/developer?utm_source=langchain&utm_medium=langchainbuild&utm_id=dev)。

**注册：**

- 使用您的电子邮件注册并验证您的注册

- 您将收到一封包含平台令牌和 OAuth 令牌的邮件，以便使用该服务。



**步骤 1：实例化工具包并获取工具：**

- 使用 cogniswitch 令牌、openAI API 密钥和 OAuth 令牌实例化 cogniswitch 工具包并获取工具。

**步骤 2：使用工具和 LLM 实例化代理：**
- 使用 cogniswitch 工具列表和 LLM 实例化代理到代理执行器中。

**步骤 3：CogniSwitch 存储工具：**

***CogniSwitch 知识源文件工具***
- 使用代理通过提供文件路径上传文件。（当前支持的格式有 .pdf、.docx、.doc、.txt、.html）  
- 文件中的内容将由 cogniswitch 处理并存储在您的知识库中。

***CogniSwitch 知识源网址工具***
- 使用代理上传网址。  
- 网址中的内容将由 cogniswitch 处理并存储在您的知识库中。

**步骤 4：CogniSwitch 状态工具：**
- 使用代理查询上传文档的状态，提供文档名称。
- 您还可以在 cogniswitch 控制台中检查文档处理的状态。

**步骤 5：CogniSwitch 答案工具：**
- 使用代理提出您的问题。
- 您将从您的知识中获得响应的答案。



```python
%pip install -qU langchain-community
```

### 导入必要的库


```python
import warnings

warnings.filterwarnings("ignore")

import os

from langchain.agents.agent_toolkits import create_conversational_retrieval_agent
from langchain_community.agent_toolkits import CogniswitchToolkit
from langchain_openai import ChatOpenAI
```

### Cogniswitch 平台令牌、OAuth 令牌和 OpenAI API 密钥


```python
cs_token = "Your CogniSwitch token"
OAI_token = "Your OpenAI API token"
oauth_token = "Your CogniSwitch authentication token"

os.environ["OPENAI_API_KEY"] = OAI_token
```

### 使用凭据实例化 cogniswitch 工具包


```python
cogniswitch_toolkit = CogniswitchToolkit(
    cs_token=cs_token, OAI_token=OAI_token, apiKey=oauth_token
)
```

### 获取 cogniswitch 工具列表


```python
tool_lst = cogniswitch_toolkit.get_tools()
```

### 实例化 llm


```python
llm = ChatOpenAI(
    temperature=0,
    openai_api_key=OAI_token,
    max_tokens=1500,
    model_name="gpt-3.5-turbo-0613",
)
```

### 创建代理执行器


```python
agent_executor = create_conversational_retrieval_agent(llm, tool_lst, verbose=False)
```

### 调用代理上传 URL


```python
response = agent_executor.invoke("upload this url https://cogniswitch.ai/developer")

print(response["output"])
```
```output
The URL https://cogniswitch.ai/developer has been uploaded successfully. The status of the document is currently being processed. You will receive an email notification once the processing is complete.
```

### 调用代理上传文件


```python
response = agent_executor.invoke("upload this file example_file.txt")

print(response["output"])
```
```output
The file example_file.txt has been uploaded successfully. The status of the document is currently being processed. You will receive an email notification once the processing is complete.
```

### 调用代理以获取文档状态


```python
response = agent_executor.invoke("Tell me the status of this document example_file.txt")

print(response["output"])
```
```output
The status of the document example_file.txt is as follows:

- Created On: 2024-01-22T19:07:42.000+00:00
- Modified On: 2024-01-22T19:07:42.000+00:00
- Document Entry ID: 153
- Status: 0 (Processing)
- Original File Name: example_file.txt
- Saved File Name: 1705950460069example_file29393011.txt

The document is currently being processed.
```

### 使用查询调用代理并获取答案

```python
response = agent_executor.invoke("CogniSwitch 如何帮助开发 GenAI 应用程序？")

print(response["output"])
```
```output
CogniSwitch can help develop GenAI applications in several ways:

1. Knowledge Extraction: CogniSwitch can extract knowledge from various sources such as documents, websites, and databases. It can analyze and store data from these sources, making it easier to access and utilize the information for GenAI applications.

2. Natural Language Processing: CogniSwitch has advanced natural language processing capabilities. It can understand and interpret human language, allowing GenAI applications to interact with users in a more conversational and intuitive manner.

3. Sentiment Analysis: CogniSwitch can analyze the sentiment of text data, such as customer reviews or social media posts. This can be useful in developing GenAI applications that can understand and respond to the emotions and opinions of users.

4. Knowledge Base Integration: CogniSwitch can integrate with existing knowledge bases or create new ones. This allows GenAI applications to access a vast amount of information and provide accurate and relevant responses to user queries.

5. Document Analysis: CogniSwitch can analyze documents and extract key information such as entities, relationships, and concepts. This can be valuable in developing GenAI applications that can understand and process large amounts of textual data.

Overall, CogniSwitch provides a range of AI-powered capabilities that can enhance the development of GenAI applications by enabling knowledge extraction, natural language processing, sentiment analysis, knowledge base integration, and document analysis.
```