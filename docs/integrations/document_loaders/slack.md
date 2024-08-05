---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/slack.ipynb
---

# Slack

>[Slack](https://slack.com/) 是一款即时通讯程序。

本笔记本涵盖如何从 `Slack` 导出的 Zip 文件加载文档。

要获取此 `Slack` 导出，请按照以下说明操作：

## 🧑 导入您自己的数据集的说明

导出您的 Slack 数据。您可以通过访问工作区管理页面并点击导入/导出选项 ({your_slack_domain}.slack.com/services/export) 来完成此操作。然后，选择正确的日期范围并点击 `开始导出`。当导出准备好时，Slack 会向您发送电子邮件和 DM。

下载将会在您的下载文件夹中生成一个 `.zip` 文件（或根据您的操作系统配置，下载位置可能会有所不同）。

复制 `.zip` 文件的路径，并将其赋值为下面的 `LOCAL_ZIPFILE`。


```python
from langchain_community.document_loaders import SlackDirectoryLoader
```


```python
# 可选设置您的 Slack URL。这将为文档源提供正确的 URL。
SLACK_WORKSPACE_URL = "https://xxx.slack.com"
LOCAL_ZIPFILE = ""  # 在此处粘贴您 Slack zip 文件的本地路径。

loader = SlackDirectoryLoader(LOCAL_ZIPFILE, SLACK_WORKSPACE_URL)
```


```python
docs = loader.load()
docs
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)