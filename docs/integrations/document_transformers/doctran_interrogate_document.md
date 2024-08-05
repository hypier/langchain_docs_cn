---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_transformers/doctran_interrogate_document.ipynb
---

# Doctran：审问文档

在向量存储知识库中使用的文档通常以叙述或对话的格式存储。然而，大多数用户查询是以问题的形式出现。如果我们在向量化之前**将文档转换为问答格式**，我们可以增加检索相关文档的可能性，并减少检索不相关文档的可能性。

我们可以使用[Doctran](https://github.com/psychic-api/doctran)库来实现这一点，该库利用OpenAI的函数调用功能来“审问”文档。

请查看[这个笔记本](https://github.com/psychic-api/doctran/blob/main/benchmark.ipynb)，了解基于原始文档与审问文档的各种查询的向量相似度分数基准。

```python
%pip install --upgrade --quiet  doctran
```

```python
import json

from langchain_community.document_transformers import DoctranQATransformer
from langchain_core.documents import Document
```

```python
from dotenv import load_dotenv

load_dotenv()
```

```output
True
```

## 输入
这是我们将要审查的文件


```python
sample_text = """[Generated with ChatGPT]

Confidential Document - For Internal Use Only

Date: July 1, 2023

Subject: Updates and Discussions on Various Topics

Dear Team,

I hope this email finds you well. In this document, I would like to provide you with some important updates and discuss various topics that require our attention. Please treat the information contained herein as highly confidential.

Security and Privacy Measures
As part of our ongoing commitment to ensure the security and privacy of our customers' data, we have implemented robust measures across all our systems. We would like to commend John Doe (email: john.doe@example.com) from the IT department for his diligent work in enhancing our network security. Moving forward, we kindly remind everyone to strictly adhere to our data protection policies and guidelines. Additionally, if you come across any potential security risks or incidents, please report them immediately to our dedicated team at security@example.com.

HR Updates and Employee Benefits
Recently, we welcomed several new team members who have made significant contributions to their respective departments. I would like to recognize Jane Smith (SSN: 049-45-5928) for her outstanding performance in customer service. Jane has consistently received positive feedback from our clients. Furthermore, please remember that the open enrollment period for our employee benefits program is fast approaching. Should you have any questions or require assistance, please contact our HR representative, Michael Johnson (phone: 418-492-3850, email: michael.johnson@example.com).

Marketing Initiatives and Campaigns
Our marketing team has been actively working on developing new strategies to increase brand awareness and drive customer engagement. We would like to thank Sarah Thompson (phone: 415-555-1234) for her exceptional efforts in managing our social media platforms. Sarah has successfully increased our follower base by 20% in the past month alone. Moreover, please mark your calendars for the upcoming product launch event on July 15th. We encourage all team members to attend and support this exciting milestone for our company.

Research and Development Projects
In our pursuit of innovation, our research and development department has been working tirelessly on various projects. I would like to acknowledge the exceptional work of David Rodriguez (email: david.rodriguez@example.com) in his role as project lead. David's contributions to the development of our cutting-edge technology have been instrumental. Furthermore, we would like to remind everyone to share their ideas and suggestions for potential new projects during our monthly R&D brainstorming session, scheduled for July 10th.

Please treat the information in this document with utmost confidentiality and ensure that it is not shared with unauthorized individuals. If you have any questions or concerns regarding the topics discussed, please do not hesitate to reach out to me directly.

Thank you for your attention, and let's continue to work together to achieve our goals.

Best regards,

Jason Fan
Cofounder & CEO
Psychic
jason@psychic.dev
"""
print(sample_text)
```
```output
[Generated with ChatGPT]

保密文件 - 仅供内部使用

日期：2023年7月1日

主题：各类主题的更新与讨论

亲爱的团队，

希望这封邮件能让你们一切安好。在这份文件中，我想向你们提供一些重要的更新，并讨论一些需要我们关注的各类主题。请将此处包含的信息视为高度保密。

安全与隐私措施
作为我们持续承诺确保客户数据安全与隐私的一部分，我们在所有系统中实施了强有力的措施。我们想对IT部门的约翰·多（电子邮件：john.doe@example.com）表示赞赏，感谢他在增强我们网络安全方面的辛勤工作。今后，我们温馨提示大家严格遵守我们的数据保护政策和指南。此外，如果您发现任何潜在的安全风险或事件，请立即向我们专门的团队报告，电子邮件为security@example.com。

人力资源更新与员工福利
最近，我们欢迎了几位新团队成员，他们在各自部门中做出了显著贡献。我想特别表彰简·史密斯（社会安全号码：049-45-5928）在客户服务方面的出色表现。简一直以来都获得了客户的积极反馈。此外，请记得我们的员工福利计划的开放注册期即将来临。如有任何问题或需要帮助，请联系我方人力资源代表迈克尔·约翰逊（电话：418-492-3850，电子邮件：michael.johnson@example.com）。

市场推广计划与活动
我们的市场团队一直在积极开发新的策略，以提高品牌知名度并推动客户参与。我们要感谢莎拉·汤普森（电话：415-555-1234）在管理我们的社交媒体平台方面的卓越努力。莎拉在过去一个月中成功地将我们的关注者数量增加了20%。此外，请在日历上标记即将于7月15日举行的产品发布活动。我们鼓励所有团队成员参加并支持这一令人振奋的公司里程碑。

研发项目
在追求创新的过程中，我们的研发部门在各类项目上辛勤工作。我想表彰大卫·罗德里格斯（电子邮件：david.rodriguez@example.com）作为项目负责人的出色工作。大卫在我们尖端技术开发方面的贡献至关重要。此外，我们想提醒大家在定于7月10日的每月研发头脑风暴会议上分享他们对潜在新项目的想法和建议。

请将本文件中的信息视为绝对保密，确保不与未经授权的个人共享。如果您对讨论的主题有任何疑问或担忧，请随时直接与我联系。

感谢您的关注，让我们继续携手实现我们的目标。

此致，

杰森·范
联合创始人兼首席执行官
Psychic
jason@psychic.dev
```

```python
documents = [Document(page_content=sample_text)]
qa_transformer = DoctranQATransformer()
transformed_document = qa_transformer.transform_documents(documents)
```

## 输出
在对文档进行询问后，结果将作为一个新文档返回，元数据中提供了问题和答案。

```python
transformed_document = qa_transformer.transform_documents(documents)
print(json.dumps(transformed_document[0].metadata, indent=2))
```
```output
{
  "questions_and_answers": [
    {
      "question": "这份文档的目的是什么？",
      "answer": "这份文档的目的是提供重要更新并讨论需要团队关注的各种主题。"
    },
    {
      "question": "如果有人发现潜在的安全风险或事件，该怎么办？",
      "answer": "如果有人发现潜在的安全风险或事件，他们应该立即向专门团队报告，邮箱是 security@example.com。"
    },
    {
      "question": "谁因增强网络安全而受到表彰？",
      "answer": "IT部门的John Doe因增强网络安全而受到表彰。"
    },
    {
      "question": "如果需要员工福利方面的帮助，应该联系谁？",
      "answer": "如需员工福利方面的帮助，应联系HR代表Michael Johnson。他的电话号码是418-492-3850，电子邮件是michael.johnson@example.com。"
    },
    {
      "question": "谁对各自部门做出了重大贡献？",
      "answer": "几位新团队成员对各自部门做出了重大贡献。"
    },
    {
      "question": "谁因在客户服务中表现出色而受到认可？",
      "answer": "Jane Smith因在客户服务中表现出色而受到认可。"
    },
    {
      "question": "谁成功增加了社交媒体上的关注者？",
      "answer": "Sarah Thompson成功增加了社交媒体上的关注者。"
    },
    {
      "question": "即将举行的产品发布会是什么时候？",
      "answer": "即将举行的产品发布会是在7月15日。"
    },
    {
      "question": "谁因作为项目负责人而获得认可？",
      "answer": "David Rodriguez因作为项目负责人而获得认可。"
    },
    {
      "question": "每月的研发头脑风暴会议定于何时？",
      "answer": "每月的研发头脑风暴会议定于7月10日。"
    }
  ]
}
```