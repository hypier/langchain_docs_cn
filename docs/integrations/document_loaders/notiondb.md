---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/notiondb.ipynb
---

# Notion DB 2/2

>[Notion](https://www.notion.so/) 是一个协作平台，支持修改后的 Markdown，集成了看板、任务、维基和数据库。它是一个全面的工作空间，用于笔记、知识和数据管理，以及项目和任务管理。

`NotionDBLoader` 是一个用于从 `Notion` 数据库加载内容的 Python 类。它从数据库中检索页面，读取其内容，并返回 Document 对象的列表。

## 要求

- 一个 `Notion` 数据库
- Notion 集成令牌

## 设置

### 1. 创建一个 Notion 表格数据库
在 Notion 中创建一个新的表格数据库。您可以向数据库添加任何列，它们将被视为元数据。例如，您可以添加以下列：

- Title: 将 Title 设置为默认属性。
- Categories: 一个多选属性，用于存储与页面相关的类别。
- Keywords: 一个多选属性，用于存储与页面相关的关键字。

将您的内容添加到数据库中每个页面的主体。NotionDBLoader 将从这些页面中提取内容和元数据。

## 2. 创建 Notion 集成
要创建 Notion 集成，请按照以下步骤操作：

1. 访问 [Notion Developers](https://www.notion.com/my-integrations) 页面，并使用您的 Notion 账户登录。
2. 点击 "+ New integration" 按钮。
3. 为您的集成命名，并选择您的数据库所在的工作区。
4. 选择所需的功能，此扩展只需要读取内容的功能。
5. 点击 "Submit" 按钮以创建集成。
创建集成后，您将获得一个 `Integration Token (API key)`。复制此令牌并妥善保管，因为您将需要它来使用 NotionDBLoader。

### 3. 将集成连接到数据库
要将您的集成连接到数据库，请按照以下步骤操作：

1. 在 Notion 中打开您的数据库。
2. 点击数据库视图右上角的三点菜单图标。
3. 点击“+ 新建集成”按钮。
4. 找到您的集成，您可能需要在搜索框中开始输入其名称。
5. 点击“连接”按钮将集成连接到数据库。

### 4. 获取数据库 ID
要获取数据库 ID，请按照以下步骤操作：

1. 在 Notion 中打开您的数据库。
2. 点击数据库视图右上角的三点菜单图标。
3. 从菜单中选择“复制链接”以将数据库 URL 复制到剪贴板。
4. 数据库 ID 是在 URL 中找到的长字符串的字母数字字符。它通常看起来像这样：https://www.notion.so/username/8935f9d140a04f95a872520c4f123456?v=.... 在这个示例中，数据库 ID 是 8935f9d140a04f95a872520c4f123456。

在数据库正确设置并且集成令牌和数据库 ID 准备好后，您现在可以使用 NotionDBLoader 代码从您的 Notion 数据库加载内容和元数据。

## 用法
NotionDBLoader 是 langchain 包中的文档加载器的一部分。您可以按如下方式使用它：

```python
from getpass import getpass

NOTION_TOKEN = getpass()
DATABASE_ID = getpass()
```
```output
········
········
```

```python
from langchain_community.document_loaders import NotionDBLoader
```


```python
loader = NotionDBLoader(
    integration_token=NOTION_TOKEN,
    database_id=DATABASE_ID,
    request_timeout_sec=30,  # 可选，默认为 10
)
```


```python
docs = loader.load()
```


```python
print(docs)
```
```output

```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)