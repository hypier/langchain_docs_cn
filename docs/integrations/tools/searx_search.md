---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/searx_search.ipynb
---

# SearxNG 搜索

本笔记本介绍如何使用自托管的 `SearxNG` 搜索 API 在网络上进行搜索。

您可以查看 [此链接](https://docs.searxng.org/dev/search_api.html) 以获取有关 `Searx API` 参数的更多信息。


```python
import pprint

from langchain_community.utilities import SearxSearchWrapper
```


```python
search = SearxSearchWrapper(searx_host="http://127.0.0.1:8888")
```

对于某些引擎，如果有直接的 `answer` 可用，包装器将打印答案，而不是完整的搜索结果列表。如果您想获取所有结果，可以使用包装器的 `results` 方法。


```python
search.run("What is the capital of France")
```



```output
'Paris is the capital of France, the largest country of Europe with 550 000 km2 (65 millions inhabitants). Paris has 2.234 million inhabitants end 2011. She is the core of Ile de France region (12 million people).'
```

## 自定义参数

SearxNG 支持 [135 个搜索引擎](https://docs.searxng.org/user/configured_engines.html)。您还可以使用任意命名的参数自定义 Searx 包装器，这些参数将传递给 Searx 搜索 API。在下面的示例中，我们将更有趣地使用来自 Searx 搜索 API 的自定义搜索参数。

在此示例中，我们将使用 `engines` 参数查询维基百科。

```python
search = SearxSearchWrapper(
    searx_host="http://127.0.0.1:8888", k=5
)  # k 是最大项目数量
```

```python
search.run("large language model ", engines=["wiki"])
```

```output
'Large language models (LLMs) represent a major advancement in AI, with the promise of transforming domains through learned knowledge. LLM sizes have been increasing 10X every year for the last few years, and as these models grow in complexity and size, so do their capabilities.\n\nGPT-3 can translate language, write essays, generate computer code, and more — all with limited to no supervision. In July 2020, OpenAI unveiled GPT-3, a language model that was easily the largest known at the time. Put simply, GPT-3 is trained to predict the next word in a sentence, much like how a text message autocomplete feature works.\n\nA large language model, or LLM, is a deep learning algorithm that can recognize, summarize, translate, predict and generate text and other content based on knowledge gained from massive datasets. Large language models are among the most successful applications of transformer models.\n\nAll of today’s well-known language models—e.g., GPT-3 from OpenAI, PaLM or LaMDA from Google, Galactica or OPT from Meta, Megatron-Turing from Nvidia/Microsoft, Jurassic-1 from AI21 Labs—are...\n\nLarge language models (LLMs) such as GPT-3are increasingly being used to generate text. These tools should be used with care, since they can generate content that is biased, non-verifiable, constitutes original research, or violates copyrights.'
```

传递其他 Searx 参数，例如 `language`。

```python
search = SearxSearchWrapper(searx_host="http://127.0.0.1:8888", k=1)
search.run("deep learning", language="es", engines=["wiki"])
```

```output
'Aprendizaje profundo (en inglés, deep learning) es un conjunto de algoritmos de aprendizaje automático (en inglés, machine learning) que intenta modelar abstracciones de alto nivel en datos usando arquitecturas computacionales que admiten transformaciones no lineales múltiples e iterativas de datos expresados en forma matricial o tensorial. 1'
```

## 使用元数据获取结果

在这个例子中，我们将使用 `categories` 参数查找科学论文，并将结果限制在 `time_range` 内（并非所有引擎都支持时间范围选项）。

我们还希望以结构化的方式获取结果，包括元数据。为此，我们将使用包装器的 `results` 方法。

```python
search = SearxSearchWrapper(searx_host="http://127.0.0.1:8888")
```

```python
results = search.results(
    "Large Language Model prompt",
    num_results=5,
    categories="science",
    time_range="year",
)
pprint.pp(results)
```
```output
[{'snippet': '… on natural language instructions, large language models (… the '
             'prompt used to steer the model, and most effective prompts … to '
             'prompt engineering, we propose Automatic Prompt …',
  'title': 'Large language models are human-level prompt engineers',
  'link': 'https://arxiv.org/abs/2211.01910',
  'engines': ['google scholar'],
  'category': 'science'},
 {'snippet': '… Large language models (LLMs) have introduced new possibilities '
             'for prototyping with AI [18]. Pre-trained on a large amount of '
             'text data, models … language instructions called prompts. …',
  'title': 'Promptchainer: Chaining large language model prompts through '
           'visual programming',
  'link': 'https://dl.acm.org/doi/abs/10.1145/3491101.3519729',
  'engines': ['google scholar'],
  'category': 'science'},
 {'snippet': '… can introspect the large prompt model. We derive the view '
             'ϕ0(X) and the model h0 from T01. However, instead of fully '
             'fine-tuning T0 during co-training, we focus on soft prompt '
             'tuning, …',
  'title': 'Co-training improves prompt-based learning for large language '
           'models',
  'link': 'https://proceedings.mlr.press/v162/lang22a.html',
  'engines': ['google scholar'],
  'category': 'science'},
 {'snippet': '… With the success of large language models (LLMs) of code and '
             'their use as … prompt design process become important. In this '
             'work, we propose a framework called Repo-Level Prompt …',
  'title': 'Repository-level prompt generation for large language models of '
           'code',
  'link': 'https://arxiv.org/abs/2206.12839',
  'engines': ['google scholar'],
  'category': 'science'},
 {'snippet': '… Figure 2 | The benefits of different components of a prompt '
             'for the largest language model (Gopher), as estimated from '
             'hierarchical logistic regression. Each point estimates the '
             'unique …',
  'title': 'Can language models learn from explanations in context?',
  'link': 'https://arxiv.org/abs/2204.02329',
  'engines': ['google scholar'],
  'category': 'science'}]
```
从 arxiv 获取论文

```python
results = search.results(
    "Large Language Model prompt", num_results=5, engines=["arxiv"]
)
pprint.pp(results)
```
```output
[{'snippet': 'Thanks to the advanced improvement of large pre-trained language '
             'models, prompt-based fine-tuning is shown to be effective on a '
             'variety of downstream tasks. Though many prompting methods have '
             'been investigated, it remains unknown which type of prompts are '
             'the most effective among three types of prompts (i.e., '
             'human-designed prompts, schema prompts and null prompts). In '
             'this work, we empirically compare the three types of prompts '
             'under both few-shot and fully-supervised settings. Our '
             'experimental results show that schema prompts are the most '
             'effective in general. Besides, the performance gaps tend to '
             'diminish when the scale of training data grows large.',
  'title': 'Do Prompts Solve NLP Tasks Using Natural Language?',
  'link': 'http://arxiv.org/abs/2203.00902v1',
  'engines': ['arxiv'],
  'category': 'science'},
 {'snippet': 'Cross-prompt automated essay scoring (AES) requires the system '
             'to use non target-prompt essays to award scores to a '
             'target-prompt essay. Since obtaining a large quantity of '
             'pre-graded essays to a particular prompt is often difficult and '
             'unrealistic, the task of cross-prompt AES is vital for the '
             'development of real-world AES systems, yet it remains an '
             'under-explored area of research. Models designed for '
             'prompt-specific AES rely heavily on prompt-specific knowledge '
             'and perform poorly in the cross-prompt setting, whereas current '
             'approaches to cross-prompt AES either require a certain quantity '
             'of labelled target-prompt essays or require a large quantity of '
             'unlabelled target-prompt essays to perform transfer learning in '
             'a multi-step manner. To address these issues, we introduce '
             'Prompt Agnostic Essay Scorer (PAES) for cross-prompt AES. Our '
             'method requires no access to labelled or unlabelled '
             'target-prompt data during training and is a single-stage '
             'approach. PAES is easy to apply in practice and achieves '
             'state-of-the-art performance on the Automated Student Assessment '
             'Prize (ASAP) dataset.',
  'title': 'Prompt Agnostic Essay Scorer: A Domain Generalization Approach to '
           'Cross-prompt Automated Essay Scoring',
  'link': 'http://arxiv.org/abs/2008.01441v1',
  'engines': ['arxiv'],
  'category': 'science'},
 {'snippet': 'Research on prompting has shown excellent performance with '
             'little or even no supervised training across many tasks. '
             'However, prompting for machine translation is still '
             'under-explored in the literature. We fill this gap by offering a '
             'systematic study on prompting strategies for translation, '
             'examining various factors for prompt template and demonstration '
             'example selection. We further explore the use of monolingual '
             'data and the feasibility of cross-lingual, cross-domain, and '
             'sentence-to-document transfer learning in prompting. Extensive '
             'experiments with GLM-130B (Zeng et al., 2022) as the testbed '
             'show that 1) the number and the quality of prompt examples '
             'matter, where using suboptimal examples degenerates translation; '
             '2) several features of prompt examples, such as semantic '
             'similarity, show significant Spearman correlation with their '
             'prompting performance; yet, none of the correlations are strong '
             'enough; 3) using pseudo parallel prompt examples constructed '
             'from monolingual data via zero-shot prompting could improve '
             'translation; and 4) improved performance is achievable by '
             'transferring knowledge from prompt examples selected in other '
             'settings. We finally provide an analysis on the model outputs '
             'and discuss several problems that prompting still suffers from.',
  'title': 'Prompting Large Language Model for Machine Translation: A Case '
           'Study',
  'link': 'http://arxiv.org/abs/2301.07069v2',
  'engines': ['arxiv'],
  'category': 'science'},
 {'snippet': 'Large language models can perform new tasks in a zero-shot '
             'fashion, given natural language prompts that specify the desired '
             'behavior. Such prompts are typically hand engineered, but can '
             'also be learned with gradient-based methods from labeled data. '
             'However, it is underexplored what factors make the prompts '
             'effective, especially when the prompts are natural language. In '
             'this paper, we investigate common attributes shared by effective '
             'prompts. We first propose a human readable prompt tuning method '
             '(F LUENT P ROMPT) based on Langevin dynamics that incorporates a '
             'fluency constraint to find a diverse distribution of effective '
             'and fluent prompts. Our analysis reveals that effective prompts '
             'are topically related to the task domain and calibrate the prior '
             'probability of label words. Based on these findings, we also '
             'propose a method for generating prompts using only unlabeled '
             'data, outperforming strong baselines by an average of 7.0% '
             'accuracy across three tasks.',
  'title': "Toward Human Readable Prompt Tuning: Kubrick's The Shining is a "
           'good movie, and a good prompt too?',
  'link': 'http://arxiv.org/abs/2212.10539v1',
  'engines': ['arxiv'],
  'category': 'science'},
 {'snippet': 'Prevailing methods for mapping large generative language models '
             "to supervised tasks may fail to sufficiently probe models' novel "
             'capabilities. Using GPT-3 as a case study, we show that 0-shot '
             'prompts can significantly outperform few-shot prompts. We '
             'suggest that the function of few-shot examples in these cases is '
             'better described as locating an already learned task rather than '
             'meta-learning. This analysis motivates rethinking the role of '
             'prompts in controlling and evaluating powerful language models. '
             'In this work, we discuss methods of prompt programming, '
             'emphasizing the usefulness of considering prompts through the '
             'lens of natural language. We explore techniques for exploiting '
             'the capacity of narratives and cultural anchors to encode '
             'nuanced intentions and techniques for encouraging deconstruction '
             'of a problem into components before producing a verdict. '
             'Informed by this more encompassing theory of prompt programming, '
             'we also introduce the idea of a metaprompt that seeds the model '
             'to generate its own natural language prompts for a range of '
             'tasks. Finally, we discuss how these more general methods of '
             'interacting with language models can be incorporated into '
             'existing and future benchmarks and practical applications.',
  'title': 'Prompt Programming for Large Language Models: Beyond the Few-Shot '
           'Paradigm',
  'link': 'http://arxiv.org/abs/2102.07350v1',
  'engines': ['arxiv'],
  'category': 'science'}]
```
在这个例子中，我们查询 `large language models` 在 `it` 类别下。然后，我们过滤来自 github 的结果。

```python
results = search.results("large language model", num_results=20, categories="it")
pprint.pp(list(filter(lambda r: r["engines"][0] == "github", results)))
```
```output
[{'snippet': '使用预训练的大型语言模型进行源代码的指南',
  'title': 'Code-LMs',
  'link': 'https://github.com/VHellendoorn/Code-LMs',
  'engines': ['github'],
  'category': 'it'},
 {'snippet': 'Dramatron使用大型语言模型生成连贯的剧本和电影剧本。',
  'title': 'dramatron',
  'link': 'https://github.com/deepmind/dramatron',
  'engines': ['github'],
  'category': 'it'}]
```
我们还可以直接查询来自`github`和其他源代码库的结果。

```python
results = search.results(
    "large language model", num_results=20, engines=["github", "gitlab"]
)
pprint.pp(results)
```
```output
[{'snippet': "《大型语言模型的水印》论文的实现，由Kirchenbauer和Geiping等人撰写。",
  'title': 'Peutlefaire / LMWatermark',
  'link': 'https://gitlab.com/BrianPulfer/LMWatermark',
  'engines': ['gitlab'],
  'category': 'it'},
 {'snippet': '使用预训练的大型语言模型进行源代码的指南',
  'title': 'Code-LMs',
  'link': 'https://github.com/VHellendoorn/Code-LMs',
  'engines': ['github'],
  'category': 'it'},
 {'snippet': '',
  'title': 'Simen Burud / 用于对话语音识别的大规模语言模型',
  'link': 'https://gitlab.com/BrianPulfer',
  'engines': ['gitlab'],
  'category': 'it'},
 {'snippet': 'Dramatron使用大型语言模型生成连贯的剧本和电影剧本。',
  'title': 'dramatron',
  'link': 'https://github.com/deepmind/dramatron',
  'engines': ['github'],
  'category': 'it'},
 {'snippet': 'loralib的代码，实现了《LoRA：大型语言模型的低秩适应》',
  'title': 'LoRA',
  'link': 'https://github.com/microsoft/LoRA',
  'engines': ['github'],
  'category': 'it'},
 {'snippet': '论文《评估在代码上训练的大型语言模型》的代码',
  'title': 'human-eval',
  'link': 'https://github.com/openai/human-eval',
  'engines': ['github'],
  'category': 'it'},
 {'snippet': '一个趋势源自《思维链提示引发大型语言模型推理》。',
  'title': 'Chain-of-ThoughtsPapers',
  'link': 'https://github.com/Timothyxxx/Chain-of-ThoughtsPapers',
  'engines': ['github'],
  'category': 'it'},
 {'snippet': 'Mistral：一种强大的西北风：透明和可访问的大规模语言模型训练框架，基于Hugging Face 🤗 Transformers构建。',
  'title': 'mistral',
  'link': 'https://github.com/stanford-crfm/mistral',
  'engines': ['github'],
  'category': 'it'},
 {'snippet': '寻找导致大型语言模型出现逆向缩放的任务的奖项',
  'title': 'prize',
  'link': 'https://github.com/inverse-scaling/prize',
  'engines': ['github'],
  'category': 'it'},
 {'snippet': 'Optimus：第一个大规模预训练的VAE语言模型',
  'title': 'Optimus',
  'link': 'https://github.com/ChunyuanLI/Optimus',
  'engines': ['github'],
  'category': 'it'},
 {'snippet': '大型语言模型研讨会（COMP790-101于UNC教堂山，2022年秋季）',
  'title': 'llm-seminar',
  'link': 'https://github.com/craffel/llm-seminar',
  'engines': ['github'],
  'category': 'it'},
 {'snippet': '一个中心开放资源，提供与大型语言模型中的思维链推理相关的数据和工具。由Samwald研究小组开发：https://samwald.info/',
  'title': 'ThoughtSource',
  'link': 'https://github.com/OpenBioLink/ThoughtSource',
  'engines': ['github'],
  'category': 'it'},
 {'snippet': '使用大型语言/多模态模型进行机器人/强化学习的论文、代码和相关网站的综合列表',
  'title': 'Awesome-LLM-Robotics',
  'link': 'https://github.com/GT-RIPL/Awesome-LLM-Robotics',
  'engines': ['github'],
  'category': 'it'},
 {'snippet': '用于大规模语言建模的生物医学训练数据策划工具',
  'title': 'biomedical',
  'link': 'https://github.com/bigscience-workshop/biomedical',
  'engines': ['github'],
  'category': 'it'},
 {'snippet': 'ChatGPT @ Home：大型语言模型（LLM）聊天机器人应用程序，由ChatGPT编写',
  'title': 'ChatGPT-at-Home',
  'link': 'https://github.com/Sentdex/ChatGPT-at-Home',
  'engines': ['github'],
  'category': 'it'},
 {'snippet': '设计和部署大型语言模型应用程序',
  'title': 'dust',
  'link': 'https://github.com/dust-tt/dust',
  'engines': ['github'],
  'category': 'it'},
 {'snippet': 'Polyglot：多语言能力均衡的大型语言模型',
  'title': 'polyglot',
  'link': 'https://github.com/EleutherAI/polyglot',
  'engines': ['github'],
  'category': 'it'},
 {'snippet': '《从大型语言模型学习视频表示》的代码发布',
  'title': 'LaViLa',
  'link': 'https://github.com/facebookresearch/LaViLa',
  'engines': ['github'],
  'category': 'it'},
 {'snippet': 'SmoothQuant：大型语言模型的准确和高效的后训练量化',
  'title': 'smoothquant',
  'link': 'https://github.com/mit-han-lab/smoothquant',
  'engines': ['github'],
  'category': 'it'},
 {'snippet': '该存储库包含论文《XL-Sum：44种语言的大规模多语言抽象摘要》的代码、数据和模型，该论文发表于计算语言学协会的发现：ACL-IJCNLP 2021。',
  'title': 'xl-sum',
  'link': 'https://github.com/csebuetnlp/xl-sum',
  'engines': ['github'],
  'category': 'it'}]
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)