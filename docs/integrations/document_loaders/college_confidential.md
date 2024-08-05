---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/college_confidential.ipynb
---

# College Confidential

>[College Confidential](https://www.collegeconfidential.com/) 提供了关于 3,800 多所大学和学院的信息。

这涵盖了如何将 `College Confidential` 网页加载到我们可以后续使用的文档格式中。


```python
from langchain_community.document_loaders import CollegeConfidentialLoader
```


```python
loader = CollegeConfidentialLoader(
    "https://www.collegeconfidential.com/colleges/brown-university/"
)
```


```python
data = loader.load()
```


```python
data
```



```output
[Document(page_content='\n\n\n\n\n\n\n\nA68FEB02-9D19-447C-B8BC-818149FD6EAF\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n                    Media (2)\n                \n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\nE45B8B13-33D4-450E-B7DB-F66EFE8F2097\n\n\n\n\n\n\n\n\n\nE45B8B13-33D4-450E-B7DB-F66EFE8F2097\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\nAbout Brown\n\n\n\n\n\n\nBrown University Overview\nBrown University 是一所位于罗德岛普罗维登斯市的私立非营利学校。布朗大学成立于 1764 年，目前每年约有 10,696 名学生入学，其中包括 7,349 名本科生。布朗大学为学生提供校园住宿。大多数学生住在校外。\n📆 标记你的日历！2023 年 1 月 5 日是提交 2023 年秋季学期申请的最后期限。\n学生在布朗大学有很多参与的方式！\n喜欢音乐或表演？加入校园乐队、合唱团，或与学校的戏剧团体一起表演。\n对新闻或传播感兴趣？布朗大学的学生可以为校园报纸撰稿、主持广播节目或担任学生主办的电视台的制作人。\n想加入兄弟会或姐妹会？布朗大学有兄弟会和姐妹会。\n计划参加体育活动？布朗大学为运动员提供了许多选择。查看所有选项，了解更多关于布朗大学学生生活的信息。\n\n\n\n2022 布朗大学概况\n\n\n\n\n\n学术日历\n其他\n\n\n总体录取率\n6%\n\n\n提前决定录取率\n16%\n\n\n提前行动录取率\n不提供 EA\n\n\n提交 SAT 成绩的申请者\n51%\n\n\n学费\n$62,680\n\n\n满足需求的百分比\n100%\n\n\n平均新生财政援助包\n$59,749\n\n\n\n\n布朗大学是一所好学校吗？\n\n不同的人对“好”学校的定义各不相同。一些可以帮助你判断什么是适合你的好学校的因素包括录取标准、录取率、学费等。\n让我们来看看这些因素，以更清楚地了解布朗大学提供的内容，以及它是否可能是适合你的大学。\n布朗大学的录取率 2022\n进入布朗大学是非常困难的。每年大约 6% 的申请者被布朗大学录取。在 2022 年，只有 2,568 名申请的 46,568 名学生被录取。\n布朗大学的保留率和毕业率\n保留率是指在一段时间内继续在学校注册的学生人数。这是了解学生对学校体验满意度以及他们是否获得成功所需支持的一个指标。\n大约 98% 的全日制新生在布朗大学开始后会在大二时返回。95% 的布朗大学本科生在六年内毕业。美国高校的平均六年毕业率为公立学校 61%，私立非营利学校 67%。\n布朗大学毕业生的就业结果\n就业安置统计数据是了解布朗大学学位价值的良好资源，因为它提供了其他毕业生的就业安置情况的视角。\n有关近期毕业生起薪的信息，请直接联系布朗大学。\n布朗大学的捐赠基金\n捐赠基金是学校投资、捐款和资产的总价值。捐赠基金不一定是学校质量的指标，但可以让你了解一所大学能够在扩展项目、改善设施和支持学生方面投入多少资金。\n截至 2022 年，布朗大学的捐赠基金总市场价值为 47 亿美元。2021 年平均大学捐赠基金为 9.05 亿美元。学校每个全日制注册学生的支出为 34,086 美元。\n布朗大学的学费和财政援助\n学费是选择大学时的另一个重要因素。一些大学可能学费较高，但在满足学生的经济需求方面做得更好。\n布朗大学满足本科生 100% 的经济需求。全日制新生的平均财政援助包大约为每年 59,749 美元。\n2022 届毕业生的平均学生债务约为每位学生 24,102 美元，不包括没有债务的学生。为了对比，将此数字与全国平均债务进行比较，后者约为每位借款人 36,000 美元。\n2023-2024 FAFSA 于 2022 年 10 月 1 日开放\n一些财政援助是基于先到先得的原则，因此请尽快填写 FAFSA。访问 FAFSA 网站以申请学生援助。请记住，FAFSA 中的第一个 F 代表免费！你不应该为提交联邦学生援助免费申请（FAFSA）支付任何费用，因此要非常小心任何向你索要钱的人。\n了解更多关于布朗大学的学费和财政援助的信息。\n根据这些信息，布朗大学看起来是否适合你？请记住，对一个人来说完美的学校可能对另一个人来说是糟糕的选择！所以问问自己：布朗大学是适合你的吗？\n如果布朗大学看起来是你想申请的学校，请点击心形按钮将其保存到你的大学列表中。\n\n仍在探索学校？\n选择以下选项以了解更多关于布朗大学的信息：\n招生\n学生生活\n学术\n学费与资助\n布朗大学社区论坛\n然后使用大学招生预测工具，通过数据科学来分析你进入一些美国最佳大学的机会。\n布朗大学在哪里？\n布朗大学位于罗德岛普罗维登斯市，距离波士顿不到一个小时。\n如果你想亲自参观布朗大学，可以计划一次访问。到达校园的最佳方式是乘坐 95 号州际公路到普罗维登斯，或预订前往最近机场 T.F. Green 的航班。\n你也可以进行虚拟校园之旅，以在不离开家门的情况下了解布朗大学和普罗维登斯的情况。\n考虑在罗德岛上学吗？\n查看罗德岛所有大学的完整列表，并将你喜欢的大学保存到你的大学列表中。\n\n\n\n大学信息\n\n\n\n\n\n\n\n\n\n                    Providence, RI 02912\n                \n\n\n\n                    校园环境：城市\n                \n\n\n\n\n\n\n\n                        (401) 863-2378\n                    \n\n                            网站\n                        \n\n                        虚拟参观\n                        \n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n布朗大学申请截止日期\n\n\n\n新生申请截止日期\n\n1 月 5 日\n\n转学申请截止日期\n\n3 月 1 日\n\n\n\n            \n                布朗大学秋季新生申请的截止日期是 \n                1 月 5 日。 \n                \n            \n          \n\n            \n                布朗大学秋季转学申请的截止日期是 \n                3 月 1 日。 \n                \n            \n          \n\n            \n            请查看学校网站 \n            以获取有关特定项目或特殊招生项目截止日期的更多信息\n            \n          \n\n\n\n\n\n\n布朗大学 ACT 成绩\n\n\n\n\nic_reflect\n\n\n\n\n\n\n\n\nACT 范围\n\n\n                  \n                    33 - 35\n                  \n                \n\n\n\nEstimated Chance of Acceptance by ACT Score\n\n\nACT Score\nEstimated Chance\n\n\n35 and Above\nGood\n\n\n33 to 35\nAvg\n\n\n33 and Less\nLow\n\n\n\n\n\n\n在你的大学申请中脱颖而出\n\n• 符合奖学金资格\n• 大多数重新考试的学生提高了他们的分数\n\n由 ACT 赞助\n\n\n            参加下一个 ACT 考试\n        \n\n\n\n\n\n布朗大学 SAT 成绩\n\n\n\n\nic_reflect\n\n\n\n\n\n\n\n\n综合 SAT 范围\n\n\n                    \n                        720 - 770\n                    \n                \n\n\n\nic_reflect\n\n\n\n\n\n\n\n\n数学 SAT 范围\n\n\n                    \n                        不可用\n                    \n                \n\n\n\nic_reflect\n\n\n\n\n\n\n\n\n阅读 SAT 范围\n\n\n                    \n                        740 - 800\n                    \n                \n\n\n\n\n\n\n        布朗大学学费和费用\n    \n\n\n\n学费和费用\n\n\n\n                        $82,286\n                    \n州内\n\n\n\n\n                        $82,286\n                    \n州外\n\n\n\n\n\n\n\n费用明细\n\n\n州内\n\n\n州外\n\n\n\n\n州内学费\n\n\n\n                            $62,680\n                        \n\n\n\n                            $62,680\n                        \n\n\n\n\n费用\n\n\n\n                            $2,466\n                        \n\n\n\n                            $2,466\n                        \n\n\n\n\n住宿\n\n\n\n                            $15,840\n                        \n\n\n\n                            $15,840\n                        \n\n\n\n\n书籍\n\n\n\n                            $1,300\n                        \n\n\n\n                            $1,300\n                        \n\n\n\n\n\n                            总计（在财政援助之前）：\n                        \n\n\n\n                            $82,286\n                        \n\n\n\n                            $82,286\n                        \n\n\n\n\n\n\n\n\n\n\n\n学生生活\n\n        想知道布朗大学的生活是怎样的？布朗大学大约有 \n        10,696 名学生注册， \n        包括 7,349 名本科生和 \n        3,347 名研究生。\n        96% 的学生全日制就读， \n        6% 的学生来自罗德岛， \n            94% 的学生来自其他州。\n    \n\n\n\n\n\n                        无\n                    \n\n\n\n\n本科注册\n\n\n\n                        96%\n                    \n全日制\n\n\n\n\n                        4%\n                    \n兼职\n\n\n\n\n\n\n\n                        94%\n                    \n\n\n\n\n居住状态\n\n\n\n                        6%\n                    \n州内\n\n\n\n\n                        94%\n                    \n州外\n\n\n\n\n\n\n\n                数据来源：IPEDS 和 Peterson\'s 数据库 © 2022 Peterson\'s LLC 保留所有权利\n            \n', lookup_str='', metadata={'source': 'https://www.collegeconfidential.com/colleges/brown-university/'}, lookup_index=0)]
```



## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)