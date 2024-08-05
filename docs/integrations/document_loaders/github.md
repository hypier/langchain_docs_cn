---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/github.ipynb
---

# GitHub

这个笔记本展示了如何加载给定仓库在 [GitHub](https://github.com/) 上的问题和拉取请求（PR）。还展示了如何加载给定仓库在 [GitHub](https://github.com/) 上的文件。我们将使用 LangChain Python 仓库作为示例。

## 设置访问令牌

要访问 GitHub API，您需要一个个人访问令牌 - 您可以在这里设置您的令牌： https://github.com/settings/tokens?type=beta。您可以将此令牌设置为环境变量 ``GITHUB_PERSONAL_ACCESS_TOKEN``，它将被自动提取，或者您可以在初始化时直接作为 ``access_token`` 命名参数传递。

```python
# If you haven't set your access token as an environment variable, pass it in here.
from getpass import getpass

ACCESS_TOKEN = getpass()
```

## 加载问题和PR


```python
from langchain_community.document_loaders import GitHubIssuesLoader
```


```python
loader = GitHubIssuesLoader(
    repo="langchain-ai/langchain",
    access_token=ACCESS_TOKEN,  # 如果您已将访问令牌设置为环境变量，请删除/注释掉此参数。
    creator="UmerHA",
)
```

让我们加载由“UmerHA”创建的所有问题和PR。

以下是您可以使用的所有过滤器列表：
- include_prs
- milestone
- state
- assignee
- creator
- mentioned
- labels
- sort
- direction
- since

有关更多信息，请参见 https://docs.github.com/en/rest/issues/issues?apiVersion=2022-11-28#list-repository-issues。


```python
docs = loader.load()
```


```python
print(docs[0].page_content)
print(docs[0].metadata)
```

## 仅加载问题

默认情况下，GitHub API 将拉取请求视为问题。要仅获取“纯”问题（即，无拉取请求），请使用 `include_prs=False`


```python
loader = GitHubIssuesLoader(
    repo="langchain-ai/langchain",
    access_token=ACCESS_TOKEN,  # delete/comment out this argument if you've set the access token as an env var.
    creator="UmerHA",
    include_prs=False,
)
docs = loader.load()
```


```python
print(docs[0].page_content)
print(docs[0].metadata)
```

## 加载 Github 文件内容

以下代码加载 `langchain-ai/langchain` 仓库中的所有 Markdown 文件


```python
from langchain.document_loaders import GithubFileLoader
```


```python
loader = GithubFileLoader(
    repo="langchain-ai/langchain",  # 仓库名称
    access_token=ACCESS_TOKEN,
    github_api_url="https://api.github.com",
    file_filter=lambda file_path: file_path.endswith(
        ".md"
    ),  # 加载所有 Markdown 文件。
)
documents = loader.load()
```

一个文档的示例输出： 

```json
documents.metadata: 
    {
      "path": "README.md",
      "sha": "82f1c4ea88ecf8d2dfsfx06a700e84be4",
      "source": "https://github.com/langchain-ai/langchain/blob/master/README.md"
    }
documents.content:
    mock content
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)