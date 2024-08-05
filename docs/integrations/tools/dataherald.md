---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/dataherald.ipynb
---

# Dataherald

本笔记本介绍了如何使用 dataherald 组件。

首先，您需要设置您的 Dataherald 账户并获取您的 API KEY：

1. 前往 dataherald 并在 [这里](https://www.dataherald.com/) 注册
2. 登录到您的管理控制台后，创建一个 API KEY
3. pip install dataherald

然后我们需要设置一些环境变量：
1. 将您的 API KEY 保存到 DATAHERALD_API_KEY 环境变量中


```python
pip install dataherald
%pip install --upgrade --quiet langchain-community
```


```python
import os

os.environ["DATAHERALD_API_KEY"] = ""
```


```python
from langchain_community.utilities.dataherald import DataheraldAPIWrapper
```


```python
dataherald = DataheraldAPIWrapper(db_connection_id="65fb766367dd22c99ce1a12d")
```


```python
dataherald.run("How many employees are in the company?")
```



```output
'select COUNT(*) from employees'
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)