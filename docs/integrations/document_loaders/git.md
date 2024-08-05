---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/git.ipynb
---

# Git

>[Git](https://en.wikipedia.org/wiki/Git) 是一个分布式版本控制系统，用于跟踪任何一组计算机文件的更改，通常用于协调程序员在软件开发过程中协作开发源代码的工作。

本笔记本演示如何从 `Git` 仓库加载文本文件。

## 从磁盘加载现有的代码库


```python
%pip install --upgrade --quiet  GitPython
```


```python
from git import Repo

repo = Repo.clone_from(
    "https://github.com/langchain-ai/langchain", to_path="./example_data/test_repo1"
)
branch = repo.head.reference
```


```python
from langchain_community.document_loaders import GitLoader
```


```python
loader = GitLoader(repo_path="./example_data/test_repo1/", branch=branch)
```


```python
data = loader.load()
```


```python
len(data)
```


```python
print(data[0])
```
```output
page_content='.venv\n.github\n.git\n.mypy_cache\n.pytest_cache\nDockerfile' metadata={'file_path': '.dockerignore', 'file_name': '.dockerignore', 'file_type': ''}
```

## 从 URL 克隆仓库


```python
from langchain_community.document_loaders import GitLoader
```


```python
loader = GitLoader(
    clone_url="https://github.com/langchain-ai/langchain",
    repo_path="./example_data/test_repo2/",
    branch="master",
)
```


```python
data = loader.load()
```


```python
len(data)
```



```output
1074
```

## 过滤要加载的文件


```python
from langchain_community.document_loaders import GitLoader

# 例如，仅加载 python 文件
loader = GitLoader(
    repo_path="./example_data/test_repo1/",
    file_filter=lambda file_path: file_path.endswith(".py"),
)
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)