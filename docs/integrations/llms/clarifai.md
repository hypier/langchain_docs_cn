---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/clarifai.ipynb
---

# Clarifai

>[Clarifai](https://www.clarifai.com/) 是一个人工智能平台，提供从数据探索、数据标注、模型训练、评估到推理的完整人工智能生命周期。

本示例介绍如何使用 LangChain 与 `Clarifai` [模型](https://clarifai.com/explore/models) 进行交互。

要使用 Clarifai，您必须拥有一个帐户和一个个人访问令牌 (PAT) 密钥。
[在这里检查](https://clarifai.com/settings/security) 获取或创建 PAT。

# 依赖

```python
# Install required dependencies
%pip install --upgrade --quiet  clarifai
```

```python
# Declare clarifai pat token as environment variable or you can pass it as argument in clarifai class.
import os

os.environ["CLARIFAI_PAT"] = "CLARIFAI_PAT_TOKEN"
```

# 导入
在这里我们将设置个人访问令牌。您可以在您的 Clarifai 账户的 [设置/安全](https://clarifai.com/settings/security) 中找到您的 PAT。

```python
# Please login and get your API key from  https://clarifai.com/settings/security
from getpass import getpass

CLARIFAI_PAT = getpass()
```

```python
# Import the required modules
from langchain.chains import LLMChain
from langchain_community.llms import Clarifai
from langchain_core.prompts import PromptTemplate
```

# 输入
创建一个用于 LLM Chain 的提示模板：

```python
template = """Question: {question}

Answer: Let's think step by step."""

prompt = PromptTemplate.from_template(template)
```

# 设置
设置模型所在的用户 ID 和应用 ID。您可以在 https://clarifai.com/explore/models 找到公共模型的列表。

您还需要初始化模型 ID，如有必要，还需初始化模型版本 ID。一些模型有多个版本，您可以选择适合您任务的版本。

或者，您可以使用 model_url（例如："https://clarifai.com/anthropic/completion/models/claude-v2"）进行初始化。


```python
USER_ID = "openai"
APP_ID = "chat-completion"
MODEL_ID = "GPT-3_5-turbo"

# You can provide a specific model version as the model_version_id arg.
# MODEL_VERSION_ID = "MODEL_VERSION_ID"
# or

MODEL_URL = "https://clarifai.com/openai/chat-completion/models/GPT-4"
```


```python
# Initialize a Clarifai LLM
clarifai_llm = Clarifai(user_id=USER_ID, app_id=APP_ID, model_id=MODEL_ID)
# or
# Initialize through Model URL
clarifai_llm = Clarifai(model_url=MODEL_URL)
```


```python
# Create LLM chain
llm_chain = LLMChain(prompt=prompt, llm=clarifai_llm)
```

# 运行链

```python
question = "What NFL team won the Super Bowl in the year Justin Beiber was born?"

llm_chain.run(question)
```

```output
' 好的，以下是解决这个问题的步骤：\n\n1. 贾斯汀·比伯出生于1994年3月1日。\n\n2. 他出生年份的超级碗是超级碗XXVIII。\n\n3. 超级碗XXVIII于1994年1月30日进行。\n\n4. 参加超级碗XXVIII的两支球队是达拉斯牛仔队和布法罗比尔队。\n\n5. 达拉斯牛仔队以30-13战胜布法罗比尔队，赢得了超级碗XXVIII。\n\n因此，贾斯汀·比伯出生年份赢得超级碗的NFL球队是达拉斯牛仔队。'
```

## 使用推理参数进行模型预测
或者，您可以使用带有推理参数（如温度、max_tokens等）的GPT模型。

```python
# 将参数初始化为字典。
params = dict(temperature=str(0.3), max_tokens=100)
```

```python
clarifai_llm = Clarifai(user_id=USER_ID, app_id=APP_ID, model_id=MODEL_ID)
llm_chain = LLMChain(
    prompt=prompt, llm=clarifai_llm, llm_kwargs={"inference_params": params}
)
```

```python
question = "可以形成多少个三位数的偶数，如果其中一个数字是5，则下一个数字必须是7？"

llm_chain.run(question)
```

```output
'步骤1：第一个数字可以是1到9之间的任何偶数，但不能是5。所以第一个数字有4个选择。\n\n步骤2：如果第一个数字不是5，则第二个数字必须是7。所以第二个数字只有1个选择。\n\n步骤3：第三个数字可以是0到9之间的任何偶数，但不能是5和7。所以有'
```

为一系列提示生成响应

```python
# 我们可以使用_generate为一系列提示生成响应。
clarifai_llm._generate(
    [
        "帮我用5句话总结美国革命的事件",
        "用幽默的方式解释火箭科学",
        "为大学运动会创建欢迎致辞的脚本",
    ],
    inference_params=params,
)
```

```output
LLMResult(generations=[[Generation(text=' 以下是美国革命关键事件的5句总结：\n\n美国革命始于美国殖民者与英国政府之间因无代表的征税问题而产生的紧张关系。1775年，英国军队与美国民兵在列克星敦和康科德爆发冲突，开启了革命战争。大陆会议任命乔治·华盛顿为大陆军的指挥官，该军队在与英国的战斗中取得了关键胜利。1776年，独立宣言被通过，正式宣布13个美国殖民地脱离英国统治。在经过数年的战斗后，革命战争于1781年在约克镇以英国的失败和美国独立的承认而结束。')], [Generation(text=" 这是对火箭科学幽默的解释：\n\n火箭科学太简单了，简直就是小孩子的游戏！只需把一个装满爆炸液体的大金属管绑在你的屁股上，点燃引信。有什么可能出错的呢？发射！嗖，你很快就会飞向月球。只要记得戴上头盔，否则你的头在离开大气层时可能会像痘痘一样爆炸。 \n\n制造火箭也很简单。只需将一些辛辣的香料、大蒜粉、辣椒粉、少许火药混合在一起，瞧，火箭燃料就做好了！如果想要额外的刺激，再加一点小苏打和醋。摇匀后倒入你的DIY苏打瓶火箭中。站远一点，看看那个宝贝飞起来！\n\n引导火箭是全家人都能享受的乐趣。只需系好安全带，按一些随机按钮，看看你会到哪里。这就像是终极惊喜假期！你永远不知道自己会到达金星、在火星上坠毁，还是快速穿越土星的环。 \n\n如果有什么问题，不用担心。火箭科学轻松简单。只需用一些胶带和疯狂胶水即时排除故障，你就能很快回到正轨。谁还需要任务控制呢？')], [Generation(text=" 这是为大学运动会准备的欢迎致辞草稿：\n\n大家早上好，欢迎来到我们学院的年度运动会！看到这么多学生、教职员工、校友和嘉宾聚集在这里，庆祝我们学院的体育精神和运动成就，真是太好了。\n\n首先，让我们感谢所有的组织者、志愿者、教练和工作人员，他们在幕后辛勤工作，使这次活动成为可能。没有你们的奉献和承诺，我们的运动会将无法举行。\n\n我还想表彰今天在场的所有运动员。你们以才华、精神和决心激励着我们。体育具有独特的力量，能够团结和激励我们的社区。通过个人和团队运动，你们展现了专注、合作、毅力和韧性，这些品质在场上和场下都会对你们大有裨益。\n\n竞争精神和公平竞争是任何体育赛事的核心价值。我鼓励大家今天积极竞争。尽全力比赛，享受乐趣。无论结果如何，都要为同伴运动员的努力和体育精神喝彩。\n\n无论胜负，这个运动会是我们建立友谊、创造终生回忆的一天。让我们把这一天变成大家的健身和友谊之日。那么，比赛开始吧。祝大家愉快！")]], llm_output=None, run=None)
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)