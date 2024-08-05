---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/google_lens.ipynb
---

# Google Lens

本笔记本介绍如何使用 Google Lens 工具获取图像信息。

首先，您需要在以下网址注册一个 `SerpApi key`： https://serpapi.com/users/sign_up。

然后，您必须使用以下命令安装 `requests`：

`pip install requests`

接下来，您需要将环境变量 `SERPAPI_API_KEY` 设置为您的 `SerpApi key`。

[或者，您可以将密钥作为参数传递给包装器 `serp_api_key="your secret key"`]

## 使用工具


```python
%pip install --upgrade --quiet  requests langchain-community
```
```output
Requirement already satisfied: requests in /opt/anaconda3/envs/langchain/lib/python3.10/site-packages (2.31.0)
Requirement already satisfied: charset-normalizer<4,>=2 in /opt/anaconda3/envs/langchain/lib/python3.10/site-packages (from requests) (3.3.2)
Requirement already satisfied: idna<4,>=2.5 in /opt/anaconda3/envs/langchain/lib/python3.10/site-packages (from requests) (3.4)
Requirement already satisfied: urllib3<3,>=1.21.1 in /opt/anaconda3/envs/langchain/lib/python3.10/site-packages (from requests) (2.1.0)
Requirement already satisfied: certifi>=2017.4.17 in /opt/anaconda3/envs/langchain/lib/python3.10/site-packages (from requests) (2023.11.17)
```

```python
import os

from langchain_community.tools.google_lens import GoogleLensQueryRun
from langchain_community.utilities.google_lens import GoogleLensAPIWrapper

os.environ["SERPAPI_API_KEY"] = ""
tool = GoogleLensQueryRun(api_wrapper=GoogleLensAPIWrapper())
```


```python
# 在丹尼·德维托的图片上运行谷歌镜头
tool.run("https://i.imgur.com/HBrB8p0.png")
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)