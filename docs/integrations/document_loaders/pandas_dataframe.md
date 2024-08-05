---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/pandas_dataframe.ipynb
---

# Pandas 数据框

本笔记本介绍如何从 [pandas](https://pandas.pydata.org/pandas-docs/stable/user_guide/index) 数据框加载数据。


```python
%pip install --upgrade --quiet  pandas
```


```python
import pandas as pd
```


```python
df = pd.read_csv("example_data/mlb_teams_2012.csv")
```


```python
df.head()
```



```html
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>球队</th>
      <th>"薪资（百万）"</th>
      <th>"胜场数"</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>国民</td>
      <td>81.34</td>
      <td>98</td>
    </tr>
    <tr>
      <th>1</th>
      <td>红人</td>
      <td>82.20</td>
      <td>97</td>
    </tr>
    <tr>
      <th>2</th>
      <td>洋基</td>
      <td>197.96</td>
      <td>95</td>
    </tr>
    <tr>
      <th>3</th>
      <td>巨人</td>
      <td>117.62</td>
      <td>94</td>
    </tr>
    <tr>
      <th>4</th>
      <td>勇士</td>
      <td>83.31</td>
      <td>94</td>
    </tr>
  </tbody>
</table>
</div> 
```



```python
from langchain_community.document_loaders import DataFrameLoader
```


```python
loader = DataFrameLoader(df, page_content_column="球队")
```


```python
loader.load()
```



```output
[Document(page_content='国民', metadata={' "薪资（百万）"': 81.34, ' "胜场数"': 98}),
 Document(page_content='红人', metadata={' "薪资（百万）"': 82.2, ' "胜场数"': 97}),
 Document(page_content='洋基', metadata={' "薪资（百万）"': 197.96, ' "胜场数"': 95}),
 Document(page_content='巨人', metadata={' "薪资（百万）"': 117.62, ' "胜场数"': 94}),
 Document(page_content='勇士', metadata={' "薪资（百万）"': 83.31, ' "胜场数"': 94}),
 Document(page_content='运动家', metadata={' "薪资（百万）"': 55.37, ' "胜场数"': 94}),
 Document(page_content='游骑兵', metadata={' "薪资（百万）"': 120.51, ' "胜场数"': 93}),
 Document(page_content='金莺', metadata={' "薪资（百万）"': 81.43, ' "胜场数"': 93}),
 Document(page_content='光芒', metadata={' "薪资（百万）"': 64.17, ' "胜场数"': 90}),
 Document(page_content='天使', metadata={' "薪资（百万）"': 154.49, ' "胜场数"': 89}),
 Document(page_content='老虎', metadata={' "薪资（百万）"': 132.3, ' "胜场数"': 88}),
 Document(page_content='红雀', metadata={' "薪资（百万）"': 110.3, ' "胜场数"': 88}),
 Document(page_content='道奇', metadata={' "薪资（百万）"': 95.14, ' "胜场数"': 86}),
 Document(page_content='白袜', metadata={' "薪资（百万）"': 96.92, ' "胜场数"': 85}),
 Document(page_content='酿酒人', metadata={' "薪资（百万）"': 97.65, ' "胜场数"': 83}),
 Document(page_content='费城人', metadata={' "薪资（百万）"': 174.54, ' "胜场数"': 81}),
 Document(page_content='响尾蛇', metadata={' "薪资（百万）"': 74.28, ' "胜场数"': 81}),
 Document(page_content='海盗', metadata={' "薪资（百万）"': 63.43, ' "胜场数"': 79}),
 Document(page_content='教士', metadata={' "薪资（百万）"': 55.24, ' "胜场数"': 76}),
 Document(page_content='水手', metadata={' "薪资（百万）"': 81.97, ' "胜场数"': 75}),
 Document(page_content='大都会', metadata={' "薪资（百万）"': 93.35, ' "胜场数"': 74}),
 Document(page_content='蓝鸟', metadata={' "薪资（百万）"': 75.48, ' "胜场数"': 73}),
 Document(page_content='皇家', metadata={' "薪资（百万）"': 60.91, ' "胜场数"': 72}),
 Document(page_content='马林鱼', metadata={' "薪资（百万）"': 118.07, ' "胜场数"': 69}),
 Document(page_content='红袜', metadata={' "薪资（百万）"': 173.18, ' "胜场数"': 69}),
 Document(page_content='印地安人', metadata={' "薪资（百万）"': 78.43, ' "胜场数"': 68}),
 Document(page_content='双城', metadata={' "薪资（百万）"': 94.08, ' "胜场数"': 66}),
 Document(page_content='洛基', metadata={' "薪资（百万）"': 78.06, ' "胜场数"': 64}),
 Document(page_content='小熊', metadata={' "薪资（百万）"': 88.19, ' "胜场数"': 61}),
 Document(page_content='太空人', metadata={' "薪资（百万）"': 60.65, ' "胜场数"': 55})]
```



```python
# 对于较大的表格使用懒加载，这样不会将完整表格读取到内存中
for i in loader.lazy_load():
    print(i)
```
```output
page_content='国民' metadata={' "薪资（百万）"': 81.34, ' "胜场数"': 98}
page_content='红人' metadata={' "薪资（百万）"': 82.2, ' "胜场数"': 97}
page_content='洋基' metadata={' "薪资（百万）"': 197.96, ' "胜场数"': 95}
page_content='巨人' metadata={' "薪资（百万）"': 117.62, ' "胜场数"': 94}
page_content='勇士' metadata={' "薪资（百万）"': 83.31, ' "胜场数"': 94}
page_content='运动家' metadata={' "薪资（百万）"': 55.37, ' "胜场数"': 94}
page_content='游骑兵' metadata={' "薪资（百万）"': 120.51, ' "胜场数"': 93}
page_content='金莺' metadata={' "薪资（百万）"': 81.43, ' "胜场数"': 93}
page_content='光芒' metadata={' "薪资（百万）"': 64.17, ' "胜场数"': 90}
page_content='天使' metadata={' "薪资（百万）"': 154.49, ' "胜场数"': 89}
page_content='老虎' metadata={' "薪资（百万）"': 132.3, ' "胜场数"': 88}
page_content='红雀' metadata={' "薪资（百万）"': 110.3, ' "胜场数"': 88}
page_content='道奇' metadata={' "薪资（百万）"': 95.14, ' "胜场数"': 86}
page_content='白袜' metadata={' "薪资（百万）"': 96.92, ' "胜场数"': 85}
page_content='酿酒人' metadata={' "薪资（百万）"': 97.65, ' "胜场数"': 83}
page_content='费城人' metadata={' "薪资（百万）"': 174.54, ' "胜场数"': 81}
page_content='响尾蛇' metadata={' "薪资（百万）"': 74.28, ' "胜场数"': 81}
page_content='海盗' metadata={' "薪资（百万）"': 63.43, ' "胜场数"': 79}
page_content='教士' metadata={' "薪资（百万）"': 55.24, ' "胜场数"': 76}
page_content='水手' metadata={' "薪资（百万）"': 81.97, ' "胜场数"': 75}
page_content='大都会' metadata={' "薪资（百万）"': 93.35, ' "胜场数"': 74}
page_content='蓝鸟' metadata={' "薪资（百万）"': 75.48, ' "胜场数"': 73}
page_content='皇家' metadata={' "薪资（百万）"': 60.91, ' "胜场数"': 72}
page_content='马林鱼' metadata={' "薪资（百万）"': 118.07, ' "胜场数"': 69}
page_content='红袜' metadata={' "薪资（百万）"': 173.18, ' "胜场数"': 69}
page_content='印地安人' metadata={' "薪资（百万）"': 78.43, ' "胜场数"': 68}
page_content='双城' metadata={' "薪资（百万）"': 94.08, ' "胜场数"': 66}
page_content='洛基' metadata={' "薪资（百万）"': 78.06, ' "胜场数"': 64}
page_content='小熊' metadata={' "薪资（百万）"': 88.19, ' "胜场数"': 61}
page_content='太空人' metadata={' "薪资（百万）"': 60.65, ' "胜场数"': 55}
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)