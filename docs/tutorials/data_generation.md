---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/tutorials/data_generation.ipynb
sidebar_class_name: hidden
---
[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/langchain-ai/langchain/blob/master/docs/docs/use_cases/data_generation.ipynb)

# 生成合成数据

合成数据是人工生成的数据，而不是从现实世界事件中收集的数据。它用于模拟真实数据，而不会侵犯隐私或遇到现实世界的限制。

合成数据的好处：

1. **隐私和安全**：没有真实的个人数据面临泄露风险。
2. **数据增强**：扩展机器学习的数据集。
3. **灵活性**：创建特定或稀有的场景。
4. **成本效益**：通常比现实世界的数据收集便宜。
5. **合规性**：帮助应对严格的数据保护法律。
6. **模型鲁棒性**：可以导致更好的通用化AI模型。
7. **快速原型制作**：无需真实数据即可快速测试。
8. **受控实验**：模拟特定条件。
9. **数据访问**：在真实数据不可用时的替代方案。

注意：尽管有这些好处，合成数据应谨慎使用，因为它可能并不总是能够捕捉现实世界的复杂性。

## 快速入门

在本笔记本中，我们将深入探讨如何使用 langchain 库生成合成医疗账单记录。当您想要开发或测试算法，但由于隐私问题或数据可用性问题不想使用真实患者数据时，这个工具特别有用。

### 设置
首先，您需要安装 langchain 库及其依赖项。由于我们使用的是 OpenAI 生成器链，因此也需要安装它。由于这是一个实验性库，我们需要在安装中包含 `langchain_experimental`。然后我们将导入必要的模块。

```python
%pip install --upgrade --quiet  langchain langchain_experimental langchain-openai
# Set env var OPENAI_API_KEY or load from a .env file:
# import dotenv
# dotenv.load_dotenv()

from langchain.prompts import FewShotPromptTemplate, PromptTemplate
from langchain_core.pydantic_v1 import BaseModel
from langchain_experimental.tabular_synthetic_data.openai import (
    OPENAI_TEMPLATE,
    create_openai_data_generator,
)
from langchain_experimental.tabular_synthetic_data.prompts import (
    SYNTHETIC_FEW_SHOT_PREFIX,
    SYNTHETIC_FEW_SHOT_SUFFIX,
)
from langchain_openai import ChatOpenAI
```

## 1. 定义您的数据模型
每个数据集都有一个结构或“模式”。下面的 MedicalBilling 类作为我们合成数据的模式。通过定义这一点，我们在向合成数据生成器说明我们期望的数据的形状和性质。

```python
class MedicalBilling(BaseModel):
    patient_id: int
    patient_name: str
    diagnosis_code: str
    procedure_code: str
    total_charge: float
    insurance_claim_amount: float
```

例如，每条记录将具有一个整数类型的 `patient_id`，一个字符串类型的 `patient_name`，等等。

## 2. 示例数据
为了指导合成数据生成器，提供一些类似真实世界的示例是很有用的。这些示例作为“种子” - 它们代表了您想要的数据类型，生成器将使用它们创建更多看起来相似的数据。

以下是一些虚构的医疗账单记录：

```python
examples = [
    {
        "example": """Patient ID: 123456, Patient Name: John Doe, Diagnosis Code: 
        J20.9, Procedure Code: 99203, Total Charge: $500, Insurance Claim Amount: $350"""
    },
    {
        "example": """Patient ID: 789012, Patient Name: Johnson Smith, Diagnosis 
        Code: M54.5, Procedure Code: 99213, Total Charge: $150, Insurance Claim Amount: $120"""
    },
    {
        "example": """Patient ID: 345678, Patient Name: Emily Stone, Diagnosis Code: 
        E11.9, Procedure Code: 99214, Total Charge: $300, Insurance Claim Amount: $250"""
    },
]
```

## 3. 创建提示模板
生成器并不是凭空知道如何创建我们的数据；我们需要引导它。我们通过创建一个提示模板来实现这一点。这个模板帮助指导底层语言模型如何以所需格式生成合成数据。

```python
OPENAI_TEMPLATE = PromptTemplate(input_variables=["example"], template="{example}")

prompt_template = FewShotPromptTemplate(
    prefix=SYNTHETIC_FEW_SHOT_PREFIX,
    examples=examples,
    suffix=SYNTHETIC_FEW_SHOT_SUFFIX,
    input_variables=["subject", "extra"],
    example_prompt=OPENAI_TEMPLATE,
)
```

`FewShotPromptTemplate` 包含：

- `prefix` 和 `suffix`：这些可能包含指导上下文或指令。
- `examples`：我们之前定义的示例数据。
- `input_variables`：这些变量（"subject", "extra"）是您可以动态填充的占位符。例如，"subject" 可能填充为 "medical_billing"，以进一步引导模型。
- `example_prompt`：这个提示模板是我们希望每个示例行在提示中采用的格式。

## 4. 创建数据生成器
在准备好模式和提示后，下一步是创建数据生成器。这个对象知道如何与底层语言模型进行通信以获取合成数据。

```python
synthetic_data_generator = create_openai_data_generator(
    output_schema=MedicalBilling,
    llm=ChatOpenAI(
        temperature=1
    ),  # You'll need to replace with your actual Language Model instance
    prompt=prompt_template,
)
```

## 5. 生成合成数据
最后，让我们获取我们的合成数据！


```python
synthetic_results = synthetic_data_generator.generate(
    subject="medical_billing",
    extra="the name must be chosen at random. Make it something you wouldn't normally choose.",
    runs=10,
)
```

此命令要求生成器生成 10 条合成的医疗账单记录。结果存储在 `synthetic_results` 中。输出将是一个 MedicalBilling pydantic 模型的列表。

### 其他实现



```python
from langchain_experimental.synthetic_data import (
    DatasetGenerator,
    create_data_generation_chain,
)
from langchain_openai import ChatOpenAI
```


```python
# LLM
model = ChatOpenAI(model="gpt-3.5-turbo", temperature=0.7)
chain = create_data_generation_chain(model)
```


```python
chain({"fields": ["blue", "yellow"], "preferences": {}})
```



```output
{'fields': ['blue', 'yellow'],
 'preferences': {},
 'text': '生动的蓝天与明亮的黄太阳形成了美丽的对比，创造出令人惊叹的色彩展示，瞬间提升了所有凝视它的人的精神。'}
```



```python
chain(
    {
        "fields": {"colors": ["blue", "yellow"]},
        "preferences": {"style": "以天气预报的风格呈现。"},
    }
)
```



```output
{'fields': {'colors': ['blue', 'yellow']},
 'preferences': {'style': '以天气预报的风格呈现。'},
 'text': "早上好！今天的天气预报为天空带来了美丽的色彩组合，蓝色和黄色的色调轻柔地融合在一起，就像一幅迷人的画作。"}
```



```python
chain(
    {
        "fields": {"actor": "Tom Hanks", "movies": ["Forrest Gump", "Green Mile"]},
        "preferences": None,
    }
)
```



```output
{'fields': {'actor': 'Tom Hanks', 'movies': ['Forrest Gump', 'Green Mile']},
 'preferences': None,
 'text': '汤姆·汉克斯，这位以其惊人多才多艺和魅力而闻名的演员，在《阿甘正传》和《绿里奇迹》等令人难忘的电影中闪耀着银幕。'}
```



```python
chain(
    {
        "fields": [
            {"actor": "Tom Hanks", "movies": ["Forrest Gump", "Green Mile"]},
            {"actor": "Mads Mikkelsen", "movies": ["Hannibal", "Another round"]},
        ],
        "preferences": {"minimum_length": 200, "style": "八卦"},
    }
)
```



```output
{'fields': [{'actor': 'Tom Hanks', 'movies': ['Forrest Gump', 'Green Mile']},
  {'actor': 'Mads Mikkelsen', 'movies': ['Hannibal', 'Another round']}],
 'preferences': {'minimum_length': 200, 'style': '八卦'},
 'text': '你知道吗，汤姆·汉克斯，这位因在《阿甘正传》和《绿里奇迹》中的角色而备受喜爱的好莱坞演员，曾与才华横溢的麦ads·米克尔森同台演出，后者因在《汉尼拔》和《另一轮》中出色的表演而获得国际认可？这两位杰出的演员将他们卓越的技能和迷人的魅力带到了大银幕上，呈现了令人难忘的表演，吸引了全世界的观众。无论是汉克斯对阿甘的动人演绎，还是米克尔森对汉尼拔·莱克特的令人毛骨悚然的刻画，这些电影都巩固了他们在电影史上的地位，对观众产生了深远的影响，使他们成为真正的银幕偶像。'}
```


可以看出，创建的示例多样化，并具备我们希望它们拥有的信息。同时，它们的风格很好地反映了给定的偏好。

## 生成示例数据集以进行提取基准测试

```python
inp = [
    {
        "Actor": "Tom Hanks",
        "Film": [
            "Forrest Gump",
            "Saving Private Ryan",
            "The Green Mile",
            "Toy Story",
            "Catch Me If You Can",
        ],
    },
    {
        "Actor": "Tom Hardy",
        "Film": [
            "Inception",
            "The Dark Knight Rises",
            "Mad Max: Fury Road",
            "The Revenant",
            "Dunkirk",
        ],
    },
]

generator = DatasetGenerator(model, {"style": "informal", "minimal length": 500})
dataset = generator(inp)
```

```python
dataset
```

```output
[{'fields': {'Actor': 'Tom Hanks',
   'Film': ['Forrest Gump',
    'Saving Private Ryan',
    'The Green Mile',
    'Toy Story',
    'Catch Me If You Can']},
  'preferences': {'style': 'informal', 'minimal length': 500},
  'text': '汤姆·汉克斯，这位多才多艺且富有魅力的演员，曾在无数经典影片中闪耀银幕，包括温暖人心且激励人心的《阿甘正传》，紧张刺激的战争剧《拯救大兵瑞恩》，情感丰富且发人深省的《绿里奇迹》，深受喜爱的动画经典《玩具总动员》，以及令人兴奋且引人入胜的真实故事改编《大追捕》。凭借他令人印象深刻的表演范围和真诚的才华，汉克斯继续吸引全球观众，给电影界留下不可磨灭的印记。'},
 {'fields': {'Actor': 'Tom Hardy',
   'Film': ['Inception',
    'The Dark Knight Rises',
    'Mad Max: Fury Road',
    'The Revenant',
    'Dunkirk']},
  'preferences': {'style': 'informal', 'minimal length': 500},
  'text': '汤姆·哈迪，这位以其强烈表演而闻名的多才多艺演员，曾在无数经典影片中闪耀银幕，包括《盗梦空间》，《黑暗骑士崛起》，《疯狂麦克斯：狂怒道》，《荒野猎人》，以及《敦刻尔克》。无论是深入潜意识的深处，还是化身臭名昭著的贝恩，亦或是在神秘的麦克斯·洛卡坦斯基身上穿越危险的荒原，哈迪对其艺术的执着始终显而易见。从他在《盗梦空间》中对无情的伊姆斯的惊人演绎，到他在《疯狂麦克斯：狂怒道》中对凶猛麦克斯的迷人转变，哈迪的多样表演范围和迷人气质吸引了观众，并在电影界留下了不可磨灭的印记。在他迄今为止最具身体挑战性的角色中，他在《荒野猎人》中忍受了冰冷荒野的严酷条件，成功塑造了粗犷的边疆人约翰·菲茨杰拉德，赢得了评论界的赞誉和奥斯卡提名。在克里斯托弗·诺兰的战争史诗《敦刻尔克》中，哈迪对皇家空军飞行员法里尔的冷静而英勇的演绎展示了他通过细腻的表演传达深刻情感的能力。凭借其像变色龙般的能力去塑造各种角色以及对艺术的坚定承诺，汤姆·哈迪无疑巩固了自己作为当代最有才华和最受追捧的演员之一的地位。'}]
```

## 从生成的示例中提取
好的，让我们看看我们是否可以从这个生成的数据中提取输出，以及它与我们的案例的比较情况！


```python
from typing import List

from langchain.chains import create_extraction_chain_pydantic
from langchain_core.output_parsers import PydanticOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_openai import OpenAI
from pydantic import BaseModel, Field
```


```python
class Actor(BaseModel):
    Actor: str = Field(description="name of an actor")
    Film: List[str] = Field(description="list of names of films they starred in")
```

### 解析器


```python
llm = OpenAI()
parser = PydanticOutputParser(pydantic_object=Actor)

prompt = PromptTemplate(
    template="从给定文本中提取字段。\n{format_instructions}\n{text}\n",
    input_variables=["text"],
    partial_variables={"format_instructions": parser.get_format_instructions()},
)

_input = prompt.format_prompt(text=dataset[0]["text"])
output = llm(_input.to_string())

parsed = parser.parse(output)
parsed
```



```output
Actor(Actor='Tom Hanks', Film=['Forrest Gump', 'Saving Private Ryan', 'The Green Mile', 'Toy Story', 'Catch Me If You Can'])
```



```python
(parsed.Actor == inp[0]["Actor"]) & (parsed.Film == inp[0]["Film"])
```



```output
True
```

### 提取器


```python
extractor = create_extraction_chain_pydantic(pydantic_schema=Actor, llm=model)
extracted = extractor.run(dataset[1]["text"])
extracted
```



```output
[Actor(Actor='Tom Hardy', Film=['Inception', 'The Dark Knight Rises', 'Mad Max: Fury Road', 'The Revenant', 'Dunkirk'])]
```



```python
(extracted[0].Actor == inp[1]["Actor"]) & (extracted[0].Film == inp[1]["Film"])
```



```output
True
```