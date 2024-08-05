---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/multimodal_inputs.ipynb
---

# 如何将多模态数据直接传递给模型

在这里，我们演示如何将多模态输入直接传递给模型。 
我们目前期望所有输入都以与 [OpenAI 期望的格式](https://platform.openai.com/docs/guides/vision) 相同的格式传递。 
对于支持多模态输入的其他模型提供者，我们在类内部添加了逻辑以转换为预期格式。

在这个例子中，我们将请求模型描述一张图片。

```python
image_url = "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg"
```

```python
from langchain_core.messages import HumanMessage
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-4o")
```

传递图像的最常见支持方式是将其作为字节字符串传递。 
这应该适用于大多数模型集成。

```python
import base64

import httpx

image_data = base64.b64encode(httpx.get(image_url).content).decode("utf-8")
```

```python
message = HumanMessage(
    content=[
        {"type": "text", "text": "描述这张图片中的天气"},
        {
            "type": "image_url",
            "image_url": {"url": f"data:image/jpeg;base64,{image_data}"},
        },
    ],
)
response = model.invoke([message])
print(response.content)
```
```output
这张图片中的天气似乎晴朗宜人。天空大部分为蓝色，散布着轻微的云朵，暗示着阳光明媚，云层很少。没有降雨或强风的迹象，整体场景看起来明亮而平静。郁郁葱葱的绿色草地和清晰的能见度进一步表明天气状况良好。
```
我们可以直接在类型为 "image_url" 的内容块中传递图像 URL。请注意，只有一些模型提供者支持此功能。

```python
message = HumanMessage(
    content=[
        {"type": "text", "text": "描述这张图片中的天气"},
        {"type": "image_url", "image_url": {"url": image_url}},
    ],
)
response = model.invoke([message])
print(response.content)
```
```output
这张图片中的天气似乎晴朗而阳光明媚。天空大部分为蓝色，散布着几朵云，暗示着能见度良好，气温可能宜人。明亮的阳光在草地和植被上投下了明显的阴影，表明很可能是白天，可能是早晨或下午。整体氛围暗示着一个温暖而宜人的日子，适合户外活动。
```
我们还可以传递多张图像。

```python
message = HumanMessage(
    content=[
        {"type": "text", "text": "这两张图片是一样的吗？"},
        {"type": "image_url", "image_url": {"url": image_url}},
        {"type": "image_url", "image_url": {"url": image_url}},
    ],
)
response = model.invoke([message])
print(response.content)
```
```output
是的，这两张图片是一样的。它们都描绘了一条木栈道延伸穿过一片草地，蓝天上有轻微的云朵。风景、光线和构图都是相同的。
```

## 工具调用

一些多模态模型也支持[工具调用](/docs/concepts/#functiontool-calling)功能。要使用这些模型调用工具，只需按照[常规方式](/docs/how_to/tool_calling)将工具绑定到它们，然后使用所需类型的内容块（例如，包含图像数据）来调用模型。

```python
from typing import Literal

from langchain_core.tools import tool


@tool
def weather_tool(weather: Literal["sunny", "cloudy", "rainy"]) -> None:
    """Describe the weather"""
    pass


model_with_tools = model.bind_tools([weather_tool])

message = HumanMessage(
    content=[
        {"type": "text", "text": "describe the weather in this image"},
        {"type": "image_url", "image_url": {"url": image_url}},
    ],
)
response = model_with_tools.invoke([message])
print(response.tool_calls)
```
```output
[{'name': 'weather_tool', 'args': {'weather': 'sunny'}, 'id': 'call_BSX4oq4SKnLlp2WlzDhToHBr'}]
```