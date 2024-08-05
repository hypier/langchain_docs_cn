---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/opaqueprompts.ipynb
---

# OpaquePrompts

[OpaquePrompts](https://opaqueprompts.readthedocs.io/en/latest/) 是一项服务，使应用程序能够利用语言模型的强大功能而不妨碍用户隐私。OpaquePrompts 旨在可组合性和易于集成到现有应用程序和服务中，可以通过一个简单的 Python 库以及 LangChain 进行使用。更重要的是，OpaquePrompts 利用 [保密计算](https://en.wikipedia.org/wiki/Confidential_computing) 的强大功能，确保即使是 OpaquePrompts 服务本身也无法访问其保护的数据。

本笔记本介绍如何使用 LangChain 与 `OpaquePrompts` 进行交互。

```python
# install the opaqueprompts and langchain packages
%pip install --upgrade --quiet  opaqueprompts langchain
```

访问 OpaquePrompts API 需要一个 API 密钥，您可以通过在 [OpaquePrompts 网站](https://opaqueprompts.opaque.co/) 上创建帐户来获取。创建帐户后，您可以在 [API 密钥页面](https:opaqueprompts.opaque.co/api-keys) 找到您的 API 密钥。

```python
import os

# Set API keys

os.environ["OPAQUEPROMPTS_API_KEY"] = "<OPAQUEPROMPTS_API_KEY>"
os.environ["OPENAI_API_KEY"] = "<OPENAI_API_KEY>"
```

# 使用 OpaquePrompts LLM 包装器

将 OpaquePrompts 应用到您的应用程序中，可以像用 OpaquePrompts 类包装您的 LLM 一样简单，只需将 `llm=OpenAI()` 替换为 `llm=OpaquePrompts(base_llm=OpenAI())`。

```python
from langchain.chains import LLMChain
from langchain.globals import set_debug, set_verbose
from langchain.memory import ConversationBufferWindowMemory
from langchain_community.llms import OpaquePrompts
from langchain_core.callbacks import StdOutCallbackHandler
from langchain_core.prompts import PromptTemplate
from langchain_openai import OpenAI

set_debug(True)
set_verbose(True)

prompt_template = """
作为 AI 助手，您将根据给定的上下文回答问题。

问题中的敏感个人信息已被屏蔽以保护隐私。
例如，如果原始文本说 "Giana is good"，它将被更改为
"PERSON_998 is good。" 

以下是如何处理这些更改：
* 将这些屏蔽的短语视为占位符，但在回答时仍要以相关的方式提及它们。
* 不同的屏蔽术语可能意味着相同的事物。
请坚持使用给定的术语，不要修改它。
* 所有屏蔽术语遵循 "TYPE_ID" 模式。
* 请不要发明新的屏蔽术语。例如，如果您看到 "PERSON_998"，请不要想出 "PERSON_997" 或 "PERSON_999"，除非它们已经在问题中。

对话历史： ```{history}```
上下文 : ```在我们最近于 2023 年 2 月 23 日上午 10:30 的会议上，John Doe 向我提供了他的个人信息。他的电子邮件是 johndoe@example.com，联系电话是 650-456-7890。他住在美国纽约市，属于美国国籍，信仰基督教，倾向于民主党。他提到他最近用他的信用卡 4111 1111 1111 1111 进行了交易，并将比特币转移到钱包地址 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa。在讨论他的欧洲旅行时，他记录下他的 IBAN 为 GB29 NWBK 6016 1331 9268 19。此外，他提供了他的网站 https://johndoeportfolio.com。John 还讨论了一些他在美国的具体信息。他说他的银行账户号码是 1234567890123456，驾照是 Y12345678。他的 ITIN 是 987-65-4321，他最近更新了他的护照，护照号码是 123456789。他强调不分享他的社会安全号码，号码是 123-45-6789。此外，他提到他通过 IP 192.168.1.1 远程访问他的工作文件，并拥有医疗执照号码 MED-123456。```
问题： ```{question}```

"""

chain = LLMChain(
    prompt=PromptTemplate.from_template(prompt_template),
    llm=OpaquePrompts(base_llm=OpenAI()),
    memory=ConversationBufferWindowMemory(k=2),
    verbose=True,
)


print(
    chain.run(
        {
            "question": """写一条消息提醒 John 为他的网站重置密码以保持安全。"""
        },
        callbacks=[StdOutCallbackHandler()],
    )
)
```

从输出中，您可以看到用户输入的上下文中包含敏感数据。

``` 
# 用户输入的上下文

在我们最近于 2023 年 2 月 23 日上午 10:30 的会议上，John Doe 向我提供了他的个人信息。他的电子邮件是 johndoe@example.com，联系电话是 650-456-7890。他住在美国纽约市，属于美国国籍，信仰基督教，倾向于民主党。他提到他最近用他的信用卡 4111 1111 1111 1111 进行了交易，并将比特币转移到钱包地址 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa。在讨论他的欧洲旅行时，他记录下他的 IBAN 为 GB29 NWBK 6016 1331 9268 19。此外，他提供了他的网站 https://johndoeportfolio.com。John 还讨论了一些他在美国的具体信息。他说他的银行账户号码是 1234567890123456，驾照是 Y12345678。他的 ITIN 是 987-65-4321，他最近更新了他的护照，护照号码是 123456789。他强调不分享他的社会安全号码，号码是 123-45-6789。此外，他提到他通过 IP 192.168.1.1 远程访问他的工作文件，并拥有医疗执照号码 MED-123456。
```

OpaquePrompts 将自动检测敏感数据并用占位符替换。

```
# OpaquePrompts 处理后的上下文

在我们最近于 DATE_TIME_3 的会议上，在 DATE_TIME_2，PERSON_3 向我提供了他的个人信息。他的电子邮件是 EMAIL_ADDRESS_1，联系电话是 PHONE_NUMBER_1。他住在 LOCATION_3，LOCATION_2，属于 NRP_3 国籍，信仰 NRP_2，倾向于民主党。他提到他最近用他的信用卡 CREDIT_CARD_1 进行了交易，并将比特币转移到钱包地址 CRYPTO_1。在讨论他的 NRP_1 旅行时，他记录下他的 IBAN 为 IBAN_CODE_1。此外，他提供了他的网站为 URL_1。PERSON_2 还讨论了一些他在 LOCATION_1 的具体信息。他说他的银行账户号码是 US_BANK_NUMBER_1，驾照是 US_DRIVER_LICENSE_2。他的 ITIN 是 US_ITIN_1，他最近更新了他的护照，护照号码是 DATE_TIME_1。他强调不分享他的社会安全号码，号码是 US_SSN_1。此外，他提到他通过 IP IP_ADDRESS_1 远程访问他的工作文件，并拥有医疗执照号码 MED-US_DRIVER_LICENSE_1。
```

占位符在 LLM 响应中使用。

```
# LLM 返回的响应

嘿 PERSON_1，只想提醒你通过你的电子邮件 EMAIL_ADDRESS_1 为你的网站 URL_1 重置密码。保持在线安全很重要，所以不要忘了去做！
```

通过用原始敏感数据替换占位符，响应被去敏感化。

```
# OpaquePrompts 去敏感化后的 LLM 响应

嘿 John，只想提醒你通过你的电子邮件 johndoe@example.com 为你的网站 https://johndoeportfolio.com 重置密码。保持在线安全很重要，所以不要忘了去做！
```

# 在 LangChain 表达式中使用 OpaquePrompts

如果现成的替代方案无法提供所需的灵活性，也可以使用与 LangChain 表达式一起使用的函数。

```python
import langchain_community.utilities.opaqueprompts as op
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

prompt = (PromptTemplate.from_template(prompt_template),)
llm = OpenAI()
pg_chain = (
    op.sanitize
    | RunnablePassthrough.assign(
        response=(lambda x: x["sanitized_input"]) | prompt | llm | StrOutputParser(),
    )
    | (lambda x: op.desanitize(x["response"], x["secure_context"]))
)

pg_chain.invoke(
    {
        "question": "Write a text message to remind John to do password reset for his website through his email to stay secure.",
        "history": "",
    }
)
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)