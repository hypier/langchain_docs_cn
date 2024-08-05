---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/azure_ai_services.ipynb
---

# Azure AI 服务

此工具包用于与 `Azure AI Services API` 交互，以实现一些多模态功能。

目前，此工具包中包含五个工具：
- **AzureAiServicesImageAnalysisTool**：用于从图像中提取标题、对象、标签和文本。
- **AzureAiServicesDocumentIntelligenceTool**：用于从文档中提取文本、表格和键值对。
- **AzureAiServicesSpeechToTextTool**：用于将语音转录为文本。
- **AzureAiServicesTextToSpeechTool**：用于将文本合成语音。
- **AzureAiServicesTextAnalyticsForHealthTool**：用于提取医疗实体。

首先，您需要设置一个 Azure 账户并创建一个 AI 服务资源。您可以按照 [这里](https://learn.microsoft.com/en-us/azure/ai-services/multi-service-resource) 的说明创建资源。

然后，您需要获取资源的端点、密钥和区域，并将它们设置为环境变量。您可以在资源的“密钥和端点”页面找到它们。

```python
%pip install --upgrade --quiet  azure-ai-formrecognizer > /dev/null
%pip install --upgrade --quiet  azure-cognitiveservices-speech > /dev/null
%pip install --upgrade --quiet  azure-ai-textanalytics > /dev/null
%pip install --upgrade --quiet  azure-ai-vision-imageanalysis > /dev/null
%pip install -qU langchain-community
```

```python
import os

os.environ["OPENAI_API_KEY"] = "sk-"
os.environ["AZURE_AI_SERVICES_KEY"] = ""
os.environ["AZURE_AI_SERVICES_ENDPOINT"] = ""
os.environ["AZURE_AI_SERVICES_REGION"] = ""
```

## 创建工具包


```python
from langchain_community.agent_toolkits import AzureAiServicesToolkit

toolkit = AzureAiServicesToolkit()
```


```python
[tool.name for tool in toolkit.get_tools()]
```



```output
['azure_ai_services_document_intelligence',
 'azure_ai_services_image_analysis',
 'azure_ai_services_speech_to_text',
 'azure_ai_services_text_to_speech',
 'azure_ai_services_text_analytics_for_health']
```

## 在代理中使用


```python
from langchain import hub
from langchain.agents import AgentExecutor, create_structured_chat_agent
from langchain_openai import OpenAI
```


```python
llm = OpenAI(temperature=0)
tools = toolkit.get_tools()
prompt = hub.pull("hwchase17/structured-chat-agent")
agent = create_structured_chat_agent(llm, tools, prompt)

agent_executor = AgentExecutor(
    agent=agent, tools=tools, verbose=True, handle_parsing_errors=True
)
```


```python
agent_executor.invoke(
    {
        "input": "我可以用这些食材做什么？ "
        + "https://images.openai.com/blob/9ad5a2ab-041f-475f-ad6a-b51899c50182/ingredients.png"
    }
)
```
```output


[1m> 进入新的 AgentExecutor 链...[0m
[32;1m[1;3m
思考：我需要使用 azure_ai_services_image_analysis 工具来分析食材的图像。
动作：
```
{
  "action": "azure_ai_services_image_analysis",
  "action_input": "https://images.openai.com/blob/9ad5a2ab-041f-475f-ad6a-b51899c50182/ingredients.png"
}
```
[0m[33;1m[1;3m标题：一组鸡蛋和碗里的面粉
对象：鸡蛋，鸡蛋，食物
标签：乳制品，成分，室内，增稠剂，食物，搅拌碗，粉末，面粉，鸡蛋，碗[0m[32;1m[1;3m
动作：
```
{
  "action": "最终答案",
  "action_input": "你可以用这些食材做蛋糕或其他烘焙食品。"
}
```

[0m

[1m> 完成链。[0m
```


```output
{'input': '我可以用这些食材做什么？ https://images.openai.com/blob/9ad5a2ab-041f-475f-ad6a-b51899c50182/ingredients.png',
 'output': '你可以用这些食材做蛋糕或其他烘焙食品。'}
```



```python
tts_result = agent_executor.invoke({"input": "告诉我一个笑话并为我朗读。"})
audio_file = tts_result.get("output")
```
```output


[1m> 进入新的 AgentExecutor 链...[0m
[32;1m[1;3m
思考：我可以使用 Azure AI 服务文本转语音 API 将文本转换为语音。
动作：
```
{
  "action": "azure_ai_services_text_to_speech",
  "action_input": "为什么科学家不相信原子？因为它们构成了一切。"
}
```
[0m[36;1m[1;3m/tmp/tmpe48vamz0.wav[0m
[32;1m[1;3m[0m

[1m> 完成链。[0m
```

```python
from IPython import display

audio = display.Audio(data=audio_file, autoplay=True, rate=22050)
display.display(audio)
```


```python
sample_input = """
患者是一名54岁的男士，过去几个月有进行性心绞痛的病史。
患者在今年7月进行了心脏导管插入术，显示右冠状动脉完全闭塞和50%的左主干病变，
有强烈的冠心病家族史，兄弟在52岁时因心肌梗死去世，另一位兄弟则为冠状动脉旁路移植术后状态。患者在2001年7月进行了压力超声心动图检查，
未发现心壁运动异常，但由于体型原因，这是一项困难的研究。患者在前外侧导联中持续六分钟，出现了轻微的ST段压低，怀疑是由于疲劳和 wrist pain，他的心绞痛等价症状。由于患者的症状加重和家族史，以及右冠状动脉完全闭塞和左主干病变的病史，建议进行开放心脏手术进行血运重建。

列出所有诊断。
"""

agent_executor.invoke({"input": sample_input})
```
```output


[1m> 进入新的 AgentExecutor 链...[0m
[32;1m[1;3m
思考：患者有进行性心绞痛的病史，强烈的冠心病家族史，以及之前的心脏导管插入术显示右冠状动脉完全闭塞和50%的左主干病变。
动作：
```
{
  "action": "azure_ai_services_text_analytics_for_health",
  "action_input": "患者是一名54岁的男士，过去几个月有进行性心绞痛的病史。患者在今年7月进行了心脏导管插入术，显示右冠状动脉完全闭塞和50%的左主干病变，有强烈的冠心病家族史，兄弟在52岁时因心肌梗死去世，另一位兄弟则为冠状动脉旁路移植术后状态。患者在2001年7月进行了压力超声心动图检查，未发现心壁运动异常，但由于体型原因，这是一项困难的研究。患者在前外侧导联中持续六分钟，出现了轻微的ST段压低，怀疑是由于疲劳和 wrist pain，他的心绞痛等价症状。由于患者的症状加重和家族史，以及右冠状动脉完全闭塞和左主干病变的病史，建议进行开放心脏手术进行血运重建。"
[0m[33;1m[1;3m文本包含以下医疗实体：54岁是年龄类型的医疗实体，男士是性别类型的医疗实体，进行性心绞痛是诊断类型的医疗实体，过去几个月是时间类型的医疗实体，心脏导管插入术是检查名称类型的医疗实体，今年7月是时间类型的医疗实体，完全是条件限定符类型的医疗实体，闭塞是症状或体征类型的医疗实体，右冠状动脉是身体结构类型的医疗实体，50是测量值类型的医疗实体，%是测量单位类型的医疗实体，左主干病变是诊断类型的医疗实体，家族是家庭关系类型的医疗实体，冠心病是诊断类型的医疗实体，兄弟是家庭关系类型的医疗实体，去世是诊断类型的医疗实体，52岁是年龄类型的医疗实体，心肌梗死是诊断类型的医疗实体，兄弟是家庭关系类型的医疗实体，冠状动脉旁路移植术是治疗名称类型的医疗实体，压力超声心动图是检查名称类型的医疗实体，2001年7月是时间类型的医疗实体，心壁运动异常是症状或体征类型的医疗实体，体型是症状或体征类型的医疗实体，六分钟是时间类型的医疗实体，轻微是条件限定符类型的医疗实体，前外侧导联中的ST段压低是症状或体征类型的医疗实体，疲劳是症状或体征类型的医疗实体，腕痛是症状或体征类型的医疗实体，心绞痛是症状或体征类型的医疗实体，增加是病程类型的医疗实体，症状是症状或体征类型的医疗实体，家族是家庭关系类型的医疗实体，左主干病变是诊断类型的医疗实体，偶尔是病程类型的医疗实体，右冠状动脉是身体结构类型的医疗实体，血运重建是治疗名称类型的医疗实体，开放心脏手术是治疗名称类型的医疗实体[0m[32;1m[1;3m
动作：
```
{
  "action": "最终答案",
  "action_input": "患者的诊断包括进行性心绞痛、右冠状动脉完全闭塞、50%的左主干病变、冠心病、心肌梗死，以及冠心病的家族史。"
}

[0m

[1m> 完成链。[0m
```


```output
{'input': "\n患者是一名54岁的男士，过去几个月有进行性心绞痛的病史。\n患者在今年7月进行了心脏导管插入术，显示右冠状动脉完全闭塞和50%的左主干病变，\n有强烈的冠心病家族史，兄弟在52岁时因心肌梗死去世，另一位兄弟则为冠状动脉旁路移植术后状态。患者在2001年7月进行了压力超声心动图检查，\n未发现心壁运动异常，但由于体型原因，这是一项困难的研究。患者在前外侧导联中持续六分钟，出现了轻微的ST段压低，怀疑是由于疲劳和腕痛，他的心绞痛等价症状。由于患者的症状加重和家族史，以及右冠状动脉完全闭塞和左主干病变的病史，建议进行开放心脏手术进行血运重建。\n\n列出所有诊断。\n",
 'output': "患者的诊断包括进行性心绞痛、右冠状动脉完全闭塞、50%的左主干病变、冠心病、心肌梗死，以及冠心病的家族史。"}
```