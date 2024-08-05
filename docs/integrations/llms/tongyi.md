---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/tongyi.ipynb
---

# 通义千问
通义千问是阿里巴巴达摩院开发的大规模语言模型。它能够通过自然语言理解和语义分析理解用户意图，基于用户的自然语言输入。它为用户在不同领域和任务中提供服务和帮助。通过提供清晰详细的指示，您可以获得更符合您期望的结果。

## 设置


```python
# 安装包
%pip install --upgrade --quiet  langchain-community dashscope
```


```python
# 获取新令牌: https://help.aliyun.com/document_detail/611472.html?spm=a2c4g.2399481.0.0
from getpass import getpass

DASHSCOPE_API_KEY = getpass()
```
```output
 ········
```

```python
import os

os.environ["DASHSCOPE_API_KEY"] = DASHSCOPE_API_KEY
```


```python
from langchain_community.llms import Tongyi
```


```python
Tongyi().invoke("在贾斯汀·比伯出生的那一年，哪支NFL球队赢得了超级碗？")
```



```output
'贾斯汀·比伯出生于1994年3月1日。同年举行的超级碗是超级碗XXVIII，于1994年1月30日举行。赢得该超级碗的是达拉斯牛仔队，他们以30-13击败了布法罗比尔队。'
```

## 链式使用


```python
from langchain_core.prompts import PromptTemplate
```


```python
llm = Tongyi()
```


```python
template = """问题: {question}

回答: 让我们一步一步来思考。"""

prompt = PromptTemplate.from_template(template)
```


```python
chain = prompt | llm
```


```python
question = "贾斯汀·比伯出生那年，哪支NFL球队赢得了超级碗？"

chain.invoke({"question": question})
```



```output
'贾斯汀·比伯出生于1994年3月1日。同年举行的超级碗是超级碗XXVIII，比赛在1994年1月30日进行。超级碗XXVIII的冠军是达拉斯牛仔队，他们以30-13战胜了布法罗比尔队。'
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)