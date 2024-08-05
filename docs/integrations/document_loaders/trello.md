---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/trello.ipynb
---

# Trello

>[Trello](https://www.atlassian.com/software/trello) 是一个基于网络的项目管理和协作工具，允许个人和团队组织和跟踪他们的任务和项目。它提供了一个称为“看板”的可视化界面，用户可以创建列表和卡片来表示他们的任务和活动。

TrelloLoader 允许您从 Trello 看板加载卡片，并基于 [py-trello](https://pypi.org/project/py-trello/) 实现。

目前仅支持 `api_key/token`。

1. 凭证生成： https://trello.com/power-ups/admin/

2. 点击手动生成令牌的链接以获取令牌。

要指定 API 密钥和令牌，您可以设置环境变量 ``TRELLO_API_KEY`` 和 ``TRELLO_TOKEN``，或者可以直接将 ``api_key`` 和 ``token`` 传递给 `from_credentials` 便利构造方法。

此加载器允许您提供看板名称，以将相应的卡片拉入 Document 对象。

请注意，看板的“名称”在官方文档中也称为“标题”：

https://support.atlassian.com/trello/docs/changing-a-boards-title-and-description/

您还可以指定多个加载参数，以包含/移除文档 page_content 属性和元数据中的不同字段。

## 功能
- 从 Trello 板加载卡片。
- 根据卡片状态（打开或关闭）过滤卡片。
- 在加载的文档中包含卡片名称、评论和检查清单。
- 自定义要包含在文档中的额外元数据字段。

默认情况下，所有卡片字段都包含在全文本 page_content 和相应的元数据中。




```python
%pip install --upgrade --quiet  py-trello beautifulsoup4 lxml
```


```python
# 如果您已经使用环境变量设置了 API 密钥和令牌，
# 可以跳过此单元并在下面的初始化步骤中注释掉 `api_key` 和 `token` 命名参数
from getpass import getpass

API_KEY = getpass()
TOKEN = getpass()
```
```output
········
········
```

```python
from langchain_community.document_loaders import TrelloLoader

# 从“Awesome Board”获取打开的卡片
loader = TrelloLoader.from_credentials(
    "Awesome Board",
    api_key=API_KEY,
    token=TOKEN,
    card_filter="open",
)
documents = loader.load()

print(documents[0].page_content)
print(documents[0].metadata)
```
```output
Review Tech partner pages
Comments:
{'title': 'Review Tech partner pages', 'id': '6475357890dc8d17f73f2dcc', 'url': 'https://trello.com/c/b0OTZwkZ/1-review-tech-partner-pages', 'labels': ['Demand Marketing'], 'list': 'Done', 'closed': False, 'due_date': ''}
```

```python
# 从“Awesome Board”获取所有卡片，但只包括
# 卡片列表（列）作为额外元数据。
loader = TrelloLoader.from_credentials(
    "Awesome Board",
    api_key=API_KEY,
    token=TOKEN,
    extra_metadata=("list"),
)
documents = loader.load()

print(documents[0].page_content)
print(documents[0].metadata)
```
```output
Review Tech partner pages
Comments:
{'title': 'Review Tech partner pages', 'id': '6475357890dc8d17f73f2dcc', 'url': 'https://trello.com/c/b0OTZwkZ/1-review-tech-partner-pages', 'list': 'Done'}
```

```python
# 从“Another Board”获取卡片，并排除卡片名称、
# 检查清单和评论的文档 page_content 文本。
loader = TrelloLoader.from_credentials(
    "test",
    api_key=API_KEY,
    token=TOKEN,
    include_card_name=False,
    include_checklist=False,
    include_comments=False,
)
documents = loader.load()

print("Document: " + documents[0].page_content)
print(documents[0].metadata)
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)