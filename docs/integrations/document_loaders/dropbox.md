---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/dropbox.ipynb
---

# Dropbox

[Dropbox](https://en.wikipedia.org/wiki/Dropbox) 是一个文件托管服务，将传统文件、云内容和网页快捷方式集中在一个地方。

本笔记本涵盖如何从 *Dropbox* 加载文档。除了文本和 PDF 文件等常见文件外，它还支持 *Dropbox Paper* 文件。

## 前提条件

1. 创建一个Dropbox应用。
2. 为应用授予以下权限范围：`files.metadata.read` 和 `files.content.read`。
3. 生成访问令牌： https://www.dropbox.com/developers/apps/create。
4. `pip install dropbox` （需要 `pip install "unstructured[pdf]"` 来支持PDF文件类型）。

## 使用说明

`DropboxLoader` 需要您创建一个 Dropbox 应用并生成访问令牌。这可以在 https://www.dropbox.com/developers/apps/create 上完成。您还需要安装 Dropbox Python SDK（pip install dropbox）。

DropboxLoader 可以从一系列 Dropbox 文件路径或单个 Dropbox 文件夹路径加载数据。两个路径都应相对于与访问令牌链接的 Dropbox 帐户的根目录。

```python
pip install dropbox
```
```output
Requirement already satisfied: dropbox in /Users/rbarragan/.local/share/virtualenvs/langchain-kv0dsrF5/lib/python3.11/site-packages (11.36.2)
Requirement already satisfied: requests>=2.16.2 in /Users/rbarragan/.local/share/virtualenvs/langchain-kv0dsrF5/lib/python3.11/site-packages (from dropbox) (2.31.0)
Requirement already satisfied: six>=1.12.0 in /Users/rbarragan/.local/share/virtualenvs/langchain-kv0dsrF5/lib/python3.11/site-packages (from dropbox) (1.16.0)
Requirement already satisfied: stone>=2 in /Users/rbarragan/.local/share/virtualenvs/langchain-kv0dsrF5/lib/python3.11/site-packages (from dropbox) (3.3.1)
Requirement already satisfied: charset-normalizer<4,>=2 in /Users/rbarragan/.local/share/virtualenvs/langchain-kv0dsrF5/lib/python3.11/site-packages (from requests>=2.16.2->dropbox) (3.2.0)
Requirement already satisfied: idna<4,>=2.5 in /Users/rbarragan/.local/share/virtualenvs/langchain-kv0dsrF5/lib/python3.11/site-packages (from requests>=2.16.2->dropbox) (3.4)
Requirement already satisfied: urllib3<3,>=1.21.1 in /Users/rbarragan/.local/share/virtualenvs/langchain-kv0dsrF5/lib/python3.11/site-packages (from requests>=2.16.2->dropbox) (2.0.4)
Requirement already satisfied: certifi>=2017.4.17 in /Users/rbarragan/.local/share/virtualenvs/langchain-kv0dsrF5/lib/python3.11/site-packages (from requests>=2.16.2->dropbox) (2023.7.22)
Requirement already satisfied: ply>=3.4 in /Users/rbarragan/.local/share/virtualenvs/langchain-kv0dsrF5/lib/python3.11/site-packages (from stone>=2->dropbox) (3.11)
Note: you may need to restart the kernel to use updated packages.
```

```python
from langchain_community.document_loaders import DropboxLoader
```


```python
# 生成访问令牌: https://www.dropbox.com/developers/apps/create.
dropbox_access_token = "<DROPBOX_ACCESS_TOKEN>"
# Dropbox 根文件夹
dropbox_folder_path = ""
```


```python
loader = DropboxLoader(
    dropbox_access_token=dropbox_access_token,
    dropbox_folder_path=dropbox_folder_path,
    recursive=False,
)
```


```python
documents = loader.load()
```
```output
File /JHSfLKn0.jpeg could not be decoded as text. Skipping.
File /A REPORT ON WILES’ CAMBRIDGE LECTURES.pdf could not be decoded as text. Skipping.
```

```python
for document in documents:
    print(document)
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)