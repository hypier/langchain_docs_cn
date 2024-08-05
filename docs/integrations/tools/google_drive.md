---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/google_drive.ipynb
---

# Google Drive

本笔记本演示如何将 LangChain 连接到 `Google Drive API`。

## 前提条件

1. 创建一个 Google Cloud 项目或使用现有项目
1. 启用 [Google Drive API](https://console.cloud.google.com/flows/enableapi?apiid=drive.googleapis.com)
1. [为桌面应用授权凭据](https://developers.google.com/drive/api/quickstart/python#authorize_credentials_for_a_desktop_application)
1. `pip install --upgrade google-api-python-client google-auth-httplib2 google-auth-oauthlib`

## 检索您的 Google Docs 数据的说明
默认情况下，`GoogleDriveTools` 和 `GoogleDriveWrapper` 期望 `credentials.json` 文件位于 `~/.credentials/credentials.json`，但可以使用 `GOOGLE_ACCOUNT_FILE` 环境变量进行配置。 
`token.json` 的位置使用相同的目录（或使用参数 `token_path`）。请注意，`token.json` 会在您第一次使用该工具时自动创建。

`GoogleDriveSearchTool` 可以通过一些请求检索一系列文件。

默认情况下，如果您使用 `folder_id`，则可以检索该文件夹内的所有文件到 `Document`，前提是名称与查询匹配。



```python
%pip install --upgrade --quiet  google-api-python-client google-auth-httplib2 google-auth-oauthlib langchain-community
```

您可以从 URL 中获取文件夹和文档 ID：

* 文件夹: https://drive.google.com/drive/u/0/folders/1yucgL9WGgWZdM1TOuKkeghlPizuzMYb5 -> 文件夹 ID 是 `"1yucgL9WGgWZdM1TOuKkeghlPizuzMYb5"`
* 文档: https://docs.google.com/document/d/1bfaMQ18_i56204VaQDVeAFpqEijJTgvurupdEDiaUQw/edit -> 文档 ID 是 `"1bfaMQ18_i56204VaQDVeAFpqEijJTgvurupdEDiaUQw"`

特殊值 `root` 是指您的个人主目录。


```python
folder_id = "root"
# folder_id='1yucgL9WGgWZdM1TOuKkeghlPizuzMYb5'
```

默认情况下，所有具有以下 MIME 类型的文件可以转换为 `Document`。
- text/text
- text/plain
- text/html
- text/csv
- text/markdown
- image/png
- image/jpeg
- application/epub+zip
- application/pdf
- application/rtf
- application/vnd.google-apps.document (GDoc)
- application/vnd.google-apps.presentation (GSlide)
- application/vnd.google-apps.spreadsheet (GSheet)
- application/vnd.google.colaboratory (Notebook colab)
- application/vnd.openxmlformats-officedocument.presentationml.presentation (PPTX)
- application/vnd.openxmlformats-officedocument.wordprocessingml.document (DOCX)

可以更新或自定义此设置。请参阅 `GoogleDriveAPIWrapper` 的文档。

但是，必须安装相应的包。


```python
%pip install --upgrade --quiet  unstructured
```


```python
from langchain_googledrive.tools.google_drive.tool import GoogleDriveSearchTool
from langchain_googledrive.utilities.google_drive import GoogleDriveAPIWrapper

# 默认情况下，仅在文件名中搜索。
tool = GoogleDriveSearchTool(
    api_wrapper=GoogleDriveAPIWrapper(
        folder_id=folder_id,
        num_results=2,
        template="gdrive-query-in-folder",  # 在文档正文中搜索
    )
)
```


```python
import logging

logging.basicConfig(level=logging.INFO)
```


```python
tool.run("machine learning")
```


```python
tool.description
```


```python
from langchain.agents import load_tools

tools = load_tools(
    ["google-drive-search"],
    folder_id=folder_id,
    template="gdrive-query-in-folder",
)
```

## 在代理中的使用


```python
from langchain.agents import AgentType, initialize_agent
from langchain_openai import OpenAI

llm = OpenAI(temperature=0)
agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION,
)
```


```python
agent.run("Search in google drive, who is 'Yann LeCun' ?")
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)