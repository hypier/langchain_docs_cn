---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/gitbook.ipynb
---

# GitBook

>[GitBook](https://docs.gitbook.com/) 是一个现代化的文档平台，团队可以在这里记录从产品到内部知识库和 API 的所有内容。

本笔记本展示了如何从任何 `GitBook` 中提取页面数据。


```python
from langchain_community.document_loaders import GitbookLoader
```

### 从单个 GitBook 页面加载


```python
loader = GitbookLoader("https://docs.gitbook.com")
```


```python
page_data = loader.load()
```


```python
page_data
```



```output
[Document(page_content='Introduction to GitBook\nGitBook 是一个现代化的文档平台，团队可以在此记录从产品到内部知识库和 API 的一切。\n我们希望帮助 \n团队更高效地工作\n，创建一个简单而强大的平台，让他们 \n分享知识\n。\n我们的使命是为每个人创造一个 \n用户友好\n和 \n协作\n的产品，通过文档创建、编辑和分享知识。\n在 5 个简单步骤中发布您的文档\n导入\n\n轻松将现有内容迁移到 GitBook。\nGit 同步\n\n利用我们与 GitHub 和 GitLab 的双向同步。\n组织您的内容\n\n创建页面和空间，并将其组织到集合中。\n协作\n\n邀请其他用户，轻松进行异步协作。\n发布您的文档\n\n与选定用户或所有人分享您的文档。\n下一步\n - 入门\n概述\n最后修改于 \n3 个月前', lookup_str='', metadata={'source': 'https://docs.gitbook.com', 'title': 'Introduction to GitBook'}, lookup_index=0)]
```

### 从给定的 GitBook 中加载所有路径
要使其正常工作，GitbookLoader 需要使用根路径（在此示例中为 `https://docs.gitbook.com`）进行初始化，并将 `load_all_paths` 设置为 `True`。

```python
loader = GitbookLoader("https://docs.gitbook.com", load_all_paths=True)
all_pages_data = loader.load()
```
```output
Fetching text from https://docs.gitbook.com/
Fetching text from https://docs.gitbook.com/getting-started/overview
Fetching text from https://docs.gitbook.com/getting-started/import
Fetching text from https://docs.gitbook.com/getting-started/git-sync
Fetching text from https://docs.gitbook.com/getting-started/content-structure
Fetching text from https://docs.gitbook.com/getting-started/collaboration
Fetching text from https://docs.gitbook.com/getting-started/publishing
Fetching text from https://docs.gitbook.com/tour/quick-find
Fetching text from https://docs.gitbook.com/tour/editor
Fetching text from https://docs.gitbook.com/tour/customization
Fetching text from https://docs.gitbook.com/tour/member-management
Fetching text from https://docs.gitbook.com/tour/pdf-export
Fetching text from https://docs.gitbook.com/tour/activity-history
Fetching text from https://docs.gitbook.com/tour/insights
Fetching text from https://docs.gitbook.com/tour/notifications
Fetching text from https://docs.gitbook.com/tour/internationalization
Fetching text from https://docs.gitbook.com/tour/keyboard-shortcuts
Fetching text from https://docs.gitbook.com/tour/seo
Fetching text from https://docs.gitbook.com/advanced-guides/custom-domain
Fetching text from https://docs.gitbook.com/advanced-guides/advanced-sharing-and-security
Fetching text from https://docs.gitbook.com/advanced-guides/integrations
Fetching text from https://docs.gitbook.com/billing-and-admin/account-settings
Fetching text from https://docs.gitbook.com/billing-and-admin/plans
Fetching text from https://docs.gitbook.com/troubleshooting/faqs
Fetching text from https://docs.gitbook.com/troubleshooting/hard-refresh
Fetching text from https://docs.gitbook.com/troubleshooting/report-bugs
Fetching text from https://docs.gitbook.com/troubleshooting/connectivity-issues
Fetching text from https://docs.gitbook.com/troubleshooting/support
```

```python
print(f"fetched {len(all_pages_data)} documents.")
# show second document
all_pages_data[2]
```
```output
fetched 28 documents.
```


```output
Document(page_content="导入\n了解如何轻松迁移您现有的文档以及支持哪些格式。\n导入功能允许您在 GitBook 中迁移和统一现有文档。您可以选择导入单个或多个页面，尽管有一些限制。\n权限\n所有具有编辑权限或更高权限的成员都可以使用导入功能。\n支持的格式\nGitBook 支持从以下网站或文件导入：\nMarkdown (.md 或 .markdown)\nHTML (.html)\nMicrosoft Word (.docx)。\n我们还支持从以下平台导入：\nConfluence\nNotion\nGitHub Wiki\nQuip\nDropbox Paper\nGoogle Docs\n您还可以在导入多个页面时上传一个包含 HTML 或 Markdown 文件的 ZIP 文件。\n注意：此功能处于测试阶段。\n欢迎您建议我们尚不支持的导入源，并告知我们\n如果您遇到任何问题。\n导入面板\n当您创建新空间时，您将有机会立即导入内容：\n新页面菜单\n通过在新页面菜单中选择 \n导入页面\n，或在目录中的页面操作菜单中选择 \n导入子页面\n来导入页面或子页面：\n通过页面操作菜单导入\n当您选择输入源时，说明将解释如何进行。\n尽管 GitBook 支持从不同类型的源导入内容，但由于产品功能和文档格式的差异，最终结果可能与您的源不同。\n限制\nGitBook 当前对导入内容有以下限制：\n单次导入中可以上传的最大页面数为 \n20。\n单次导入中可以上传的最大文件数（图像等）为 \n20。\n开始使用 - \n上一个\n概述\n下一个\n - 开始使用\nGit 同步\n最后修改 \n4 个月前", lookup_str='', metadata={'source': 'https://docs.gitbook.com/getting-started/import', 'title': '导入'}, lookup_index=0)
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)