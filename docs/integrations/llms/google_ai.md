---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/google_ai.ipynb
keywords: [gemini, GoogleGenerativeAI, gemini-pro]
---

# Google AI


:::caution
您当前正在查看有关使用 Google 模型作为 [文本补全模型](/docs/concepts/#llms) 的文档。许多流行的 Google 模型是 [聊天补全模型](/docs/concepts/#chat-models)。

您可能想查看 [此页面](/docs/integrations/chat/google_generative_ai/)。
:::

关于如何使用 [Google Generative AI](https://developers.generativeai.google/) 模型与 Langchain 的指南。注意：它与 Google Cloud Vertex AI [集成](/docs/integrations/llms/google_vertex_ai_palm) 是分开的。

## 设置

要使用 Google Generative AI，您必须安装 `langchain-google-genai` Python 包并生成 API 密钥。[阅读更多详情](https://developers.generativeai.google/)。

```python
%pip install --upgrade --quiet  langchain-google-genai
```

```python
from langchain_google_genai import GoogleGenerativeAI
```

```python
from getpass import getpass

api_key = getpass()
```

```python
llm = GoogleGenerativeAI(model="models/text-bison-001", google_api_key=api_key)
print(
    llm.invoke(
        "What are some of the pros and cons of Python as a programming language?"
    )
)
```
```output
**Python 的优点：**

* **易于学习：** Python 是一种非常易于学习的编程语言，即使是初学者也能轻松上手。它的语法简单明了，有很多资源可以帮助您入门。
* **多功能：** Python 可用于多种任务，包括网页开发、数据科学和机器学习。它也是初学者的一个不错选择，因为可以用于多种项目，您可以先学习基础知识，然后再进行更复杂的任务。
* **高级：** Python 是一种高级编程语言，这意味着它比其他编程语言更接近人类语言。这使得它更容易阅读和理解，这对初学者来说是一个很大的优势。
* **开源：** Python 是一种开源编程语言，这意味着它可以免费使用，并且有很多资源可供学习。
* **社区：** Python 拥有一个庞大且活跃的开发者社区，这意味着如果您遇到困难，会有很多人可以帮助您。

**Python 的缺点：**

* **速度慢：** 与一些其他语言（如 C++）相比，Python 是一种相对较慢的编程语言。如果您正在处理计算密集型任务，这可能是一个缺点。
* **性能不如其他语言：** Python 的性能不如一些其他编程语言（如 C++ 或 Java）。如果您正在进行需要高性能的项目，这可能是一个缺点。
* **动态类型：** Python 是一种动态类型的编程语言，这意味着变量的类型可以在运行时改变。如果您需要确保代码的类型安全，这可能是一个缺点。
* **内存管理不当：** Python 使用垃圾回收系统来管理内存。如果您需要对内存管理有更多控制，这可能是一个缺点。

总的来说，Python 是一个非常适合初学者的编程语言。它易于学习、多功能，并且拥有一个庞大的开发者社区。然而，了解它的局限性，如速度慢和性能不足，也是很重要的。
```

```python
llm = GoogleGenerativeAI(model="gemini-pro", google_api_key=api_key)
print(
    llm.invoke(
        "What are some of the pros and cons of Python as a programming language?"
    )
)
```
```output
**优点：**

* **简单易读：** Python 以其简单易读的语法而闻名，这使得初学者能够轻松上手，并减少错误的可能性。它使用缩进来定义代码块，使代码结构清晰且视觉上美观。

* **多功能性：** Python 是一种通用语言，这意味着它可以用于广泛的任务，包括网页开发、数据科学、机器学习和桌面应用程序。这种多功能性使其成为各种项目和行业的热门选择。

* **庞大的社区：** Python 拥有一个庞大且活跃的开发者社区，这为其增长和普及做出了贡献。这个社区提供了丰富的文档、教程和开源库，使 Python 开发者能够轻松找到支持和资源。

* **丰富的库：** Python 提供了丰富的库和框架，用于各种任务，例如数据分析（NumPy、Pandas）、网页开发（Django、Flask）、机器学习（Scikit-learn、TensorFlow）等。这些库提供了预构建的函数和模块，使开发者能够快速高效地解决常见问题。

* **跨平台支持：** Python 是跨平台的，这意味着它可以在各种操作系统上运行，包括 Windows、macOS 和 Linux。这使得开发者能够编写可以轻松共享和在不同平台上使用的代码。

**缺点：**

* **速度和性能：** 由于其解释性特性，Python 通常比 C++ 或 Java 等编译语言慢。这在性能密集型任务（如实时系统或大量数值计算）中可能是一个缺点。

* **内存使用：** 与编译语言相比，Python 程序往往消耗更多内存。这是因为 Python 使用动态内存分配系统，这可能导致内存碎片化和更高的内存使用。

* **缺乏静态类型：** Python 是一种动态类型语言，这意味着变量的数据类型没有明确的定义。这可能使得在开发过程中检测类型错误变得具有挑战性，从而导致运行时出现意外行为或错误。

* **全局解释器锁（GIL）：** Python 使用全局解释器锁（GIL）来确保一次只能有一个线程执行 Python 字节码。这可能限制 Python 程序的可扩展性和并行性，特别是在多线程或多进程场景中。

* **包管理：** 尽管 Python 拥有丰富的库和包生态系统，但管理依赖关系和包版本可能会很具挑战性。Python 包索引（PyPI）是 Python 包的官方存储库，但确保兼容性并避免不同版本包之间的冲突可能会很困难。
```

## 在链中使用


```python
from langchain_core.prompts import PromptTemplate
```


```python
template = """Question: {question}

Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)

chain = prompt | llm

question = "How much is 2+2?"
print(chain.invoke({"question": question}))
```
```output
4
```

## 流式调用


```python
import sys

for chunk in llm.stream("Tell me a short poem about snow"):
    sys.stdout.write(chunk)
    sys.stdout.flush()
```
```output
In winter's embrace, a silent ballet,
Snowflakes descend, a celestial display.
Whispering secrets, they softly fall,
A blanket of white, covering all.

With gentle grace, they paint the land,
Transforming the world into a winter wonderland.
Trees stand adorned in icy splendor,
A glistening spectacle, a sight to render.

Snowflakes twirl, like dancers on a stage,
Creating a symphony, a winter montage.
Their silent whispers, a sweet serenade,
As they dance and twirl, a snowy cascade.

In the hush of dawn, a frosty morn,
Snow sparkles bright, like diamonds reborn.
Each flake unique, in its own design,
A masterpiece crafted by the divine.

So let us revel in this wintry bliss,
As snowflakes fall, with a gentle kiss.
For in their embrace, we find a peace profound,
A frozen world, with magic all around.
```

### 安全设置

Gemini 模型具有默认的安全设置，可以被覆盖。如果您从模型中收到大量“安全警告”，可以尝试调整模型的 `safety_settings` 属性。例如，要关闭对危险内容的安全阻止，可以按如下方式构造您的 LLM：

```python
from langchain_google_genai import GoogleGenerativeAI, HarmBlockThreshold, HarmCategory

llm = GoogleGenerativeAI(
    model="gemini-pro",
    google_api_key=api_key,
    safety_settings={
        HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT: HarmBlockThreshold.BLOCK_NONE,
    },
)
```

有关可用类别和阈值的枚举，请参阅谷歌的 [安全设置类型](https://ai.google.dev/api/python/google/generativeai/types/SafetySettingDict)。

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)