---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/azure_cognitive_services.ipynb
---

# Azure Cognitive Services

此工具包用于与 `Azure Cognitive Services API` 交互，以实现一些多模态功能。

目前，此工具包中捆绑了四个工具：
- AzureCogsImageAnalysisTool：用于从图像中提取标题、对象、标签和文本。（注意：由于依赖于仅在 Windows 和 Linux 上支持的 `azure-ai-vision` 包，此工具在 Mac OS 上尚不可用。）
- AzureCogsFormRecognizerTool：用于从文档中提取文本、表格和键值对。
- AzureCogsSpeech2TextTool：用于将语音转录为文本。
- AzureCogsText2SpeechTool：用于将文本合成语音。
- AzureCogsTextAnalyticsHealthTool：用于提取医疗实体。

首先，您需要设置一个 Azure 帐户并创建一个认知服务资源。您可以按照 [这里](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-apis-create-account?tabs=multiservice%2Cwindows) 的说明创建资源。

然后，您需要获取资源的端点、密钥和区域，并将它们设置为环境变量。您可以在资源的“密钥和端点”页面找到它们。

```python
%pip install --upgrade --quiet  azure-ai-formrecognizer > /dev/null
%pip install --upgrade --quiet  azure-cognitiveservices-speech > /dev/null
%pip install --upgrade --quiet  azure-ai-textanalytics > /dev/null

# For Windows/Linux
%pip install --upgrade --quiet  azure-ai-vision > /dev/null
```

```python
%pip install -qU langchain-community
```

```python
import os

os.environ["OPENAI_API_KEY"] = "sk-"
os.environ["AZURE_COGS_KEY"] = ""
os.environ["AZURE_COGS_ENDPOINT"] = ""
os.environ["AZURE_COGS_REGION"] = ""
```

## 创建工具包


```python
from langchain_community.agent_toolkits import AzureCognitiveServicesToolkit

toolkit = AzureCognitiveServicesToolkit()
```


```python
[tool.name for tool in toolkit.get_tools()]
```



```output
['Azure Cognitive Services Image Analysis',
 'Azure Cognitive Services Form Recognizer',
 'Azure Cognitive Services Speech2Text',
 'Azure Cognitive Services Text2Speech']
```

## 在代理中使用


```python
from langchain.agents import AgentType, initialize_agent
from langchain_openai import OpenAI
```


```python
llm = OpenAI(temperature=0)
agent = initialize_agent(
    tools=toolkit.get_tools(),
    llm=llm,
    agent=AgentType.STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
)
```


```python
agent.run(
    "我可以用这些食材做些什么？"
    "https://images.openai.com/blob/9ad5a2ab-041f-475f-ad6a-b51899c50182/ingredients.png"
)
```
```output


[1m> 进入新的 AgentExecutor 链...[0m
[32;1m[1;3m
动作:
```
{
  "action": "Azure Cognitive Services Image Analysis",
  "action_input": "https://images.openai.com/blob/9ad5a2ab-041f-475f-ad6a-b51899c50182/ingredients.png"
}
```

[0m
观察: [36;1m[1;3m标题: 一组鸡蛋和碗里的面粉
物体: 鸡蛋, 鸡蛋, 食物
标签: 乳制品, 食材, 室内, 增稠剂, 食物, 混合碗, 粉末, 面粉, 鸡蛋, 碗[0m
思考:[32;1m[1;3m 我可以利用这些物体和标签来建议食谱
动作:
```
{
  "action": "最终答案",
  "action_input": "你可以用这些食材做煎饼、煎蛋卷或咸派！"
}
```[0m

[1m> 完成链。[0m
```


```output
'你可以用这些食材做煎饼、煎蛋卷或咸派！'
```



```python
audio_file = agent.run("给我讲个笑话并为我朗读。")
```
```output


[1m> 进入新的 AgentExecutor 链...[0m
[32;1m[1;3m动作:
```
{
  "action": "Azure Cognitive Services Text2Speech",
  "action_input": "为什么鸡要穿过游乐场？为了到达另一边的滑梯！"
}
```

[0m
观察: [31;1m[1;3m/tmp/tmpa3uu_j6b.wav[0m
思考:[32;1m[1;3m 我有笑话的音频文件
动作:
```
{
  "action": "最终答案",
  "action_input": "/tmp/tmpa3uu_j6b.wav"
}
```[0m

[1m> 完成链。[0m
```


```output
'/tmp/tmpa3uu_j6b.wav'
```



```python
from IPython import display

audio = display.Audio(audio_file)
display.display(audio)
```


```python
agent.run(
    """该患者是一位54岁的男士，过去几个月有渐进性心绞痛的病史。
该患者在今年7月进行了心脏导管插入术，结果显示右冠状动脉完全闭塞，左主干病变50%,
并且有强烈的冠心病家族史，兄弟在52岁时因心肌梗死去世，
另一个兄弟则是冠状动脉旁路移植术后。该患者在2001年7月进行了压力超声心动图检查，
结果显示没有壁运动异常，但由于体型原因，这是一项困难的检查。该患者在前外侧导联中有最小的ST段压低，持续了六分钟，认为是由于疲劳和手腕疼痛引起的，属于他的心绞痛等价症状。由于患者症状加重、家族史和左主干病变的历史以及右冠状动脉的完全闭塞，建议进行开放心脏手术的血运重建。

列出所有诊断。
"""
)
```
```output


[1m> 进入新的 AgentExecutor 链...[0m
[32;1m[1;3m动作:
```
{
  "action": "azure_cognitive_services_text_analyics_health",
  "action_input": "该患者是一位54岁的男士，过去几个月有渐进性心绞痛的病史。该患者在今年7月进行了心脏导管插入术，结果显示右冠状动脉完全闭塞，左主干病变50%，并且有强烈的冠心病家族史，兄弟在52岁时因心肌梗死去世，另一个兄弟则是冠状动脉旁路移植术后。该患者在2001年7月进行了压力超声心动图检查，结果显示没有壁运动异常，但由于体型原因，这是一项困难的检查。该患者在前外侧导联中有最小的ST段压低，持续了六分钟，认为是由于疲劳和手腕疼痛引起的，属于他的心绞痛等价症状。由于患者症状加重、家族史和左主干病变的历史以及右冠状动脉的完全闭塞，建议进行开放心脏手术的血运重建。"
}
```
[0m
观察: [36;1m[1;3m该文本包含以下医疗实体: 54岁是年龄类型的医疗实体，男士是性别类型的医疗实体，渐进性心绞痛是诊断类型的医疗实体，过去几个月是时间类型的医疗实体，心脏导管插入术是检查名称类型的医疗实体，今年7月是时间类型的医疗实体，完全是条件限定词类型的医疗实体，闭塞是症状或体征类型的医疗实体，右冠状动脉是身体结构类型的医疗实体，50是测量值类型的医疗实体，%是测量单位类型的医疗实体，左主干是身体结构类型的医疗实体，疾病是诊断类型的医疗实体，家族是家庭关系类型的医疗实体，冠心病是诊断类型的医疗实体，兄弟是家庭关系类型的医疗实体，去世是诊断类型的医疗实体，52是年龄类型的医疗实体，心肌梗死是诊断类型的医疗实体，兄弟是家庭关系类型的医疗实体，冠状动脉旁路移植术是治疗名称类型的医疗实体，压力超声心动图是检查名称类型的医疗实体，2001年7月是时间类型的医疗实体，壁运动异常是症状或体征类型的医疗实体，体型是症状或体征类型的医疗实体，六分钟是时间类型的医疗实体，最小是条件限定词类型的医疗实体，前外侧导联中的ST段压低是症状或体征类型的医疗实体，疲劳是症状或体征类型的医疗实体，手腕疼痛是症状或体征类型的医疗实体，心绞痛等价症状是症状或体征类型的医疗实体，增加是病程类型的医疗实体，症状是症状或体征类型的医疗实体，家族是家庭关系类型的医疗实体，左是方向类型的医疗实体，主干是身体结构类型的医疗实体，疾病是诊断类型的医疗实体，偶尔是病程类型的医疗实体，右冠状动脉是身体结构类型的医疗实体，血运重建是治疗名称类型的医疗实体，开放心脏手术是治疗名称类型的医疗实体[0m
思考:[32;1m[1;3m 我知道该如何回应
动作:
```
{
  "action": "最终答案",
  "action_input": "该文本包含以下诊断: 渐进性心绞痛、冠心病、心肌梗死和冠状动脉旁路移植术。"
}
```[0m

[1m> 完成链。[0m
```


```output
'该文本包含以下诊断: 渐进性心绞痛、冠心病、心肌梗死和冠状动脉旁路移植术。'
```