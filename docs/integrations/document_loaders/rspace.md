---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/rspace.ipynb
---
本笔记本展示了如何使用 RSpace 文档加载器将研究笔记和文档从 RSpace 电子实验室笔记本导入 Langchain 流水线。

首先，您需要一个 RSpace 账户和一个 API 密钥。

您可以在 [https://community.researchspace.com](https://community.researchspace.com) 注册一个免费账户，或者使用您的机构 RSpace。

您可以在账户的个人资料页面获取 RSpace API 令牌。 

```python
%pip install --upgrade --quiet  rspace_client
```

最好将您的 RSpace API 密钥存储为环境变量。 

    RSPACE_API_KEY=<YOUR_KEY>

您还需要设置 RSpace 安装的 URL，例如：

    RSPACE_URL=https://community.researchspace.com

如果您使用这些确切的环境变量名称，它们将被自动检测到。 

```python
from langchain_community.document_loaders.rspace import RSpaceLoader
```

您可以从 RSpace 导入各种项目：

* 单个 RSpace 结构化或基本文档。这将与 Langchain 文档 1-1 映射。
* 一个文件夹或笔记本。笔记本或文件夹中的所有文档将作为 Langchain 文档导入。
* 如果您在 RSpace 画廊中有 PDF 文件，这些也可以单独导入。在后台，Langchain 的 PDF 加载器将被使用，这会为每个 PDF 页面创建一个 Langchain 文档。 

```python
## 将这些 ID 替换为您自己研究笔记中的一些 ID。
## 确保使用全局 ID（带有两个字符的前缀）。这有助于加载器知道要进行哪些 API 调用
## 到 RSpace API。

rspace_ids = ["NB1932027", "FL1921314", "SD1932029", "GL1932384"]
for rs_id in rspace_ids:
    loader = RSpaceLoader(global_id=rs_id)
    docs = loader.load()
    for doc in docs:
        ## 名称和 ID 被添加到 'source' 元数据属性中。
        print(doc.metadata)
        print(doc.page_content[:500])
```

如果您不想像上面那样使用环境变量，您可以将这些参数传递给 RSpaceLoader。

```python
loader = RSpaceLoader(
    global_id=rs_id, api_key="MY_API_KEY", url="https://my.researchspace.com"
)
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)