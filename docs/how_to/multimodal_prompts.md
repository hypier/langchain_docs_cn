---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/multimodal_prompts.ipynb
---

# 如何使用多模态提示

在这里，我们演示如何使用提示模板来格式化模型的多模态输入。

在这个例子中，我们将请求一个模型描述一张图像。

```python
import base64

import httpx

image_url = "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg"
image_data = base64.b64encode(httpx.get(image_url).content).decode("utf-8")
```

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-4o")
```

```python
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "描述提供的图像"),
        (
            "user",
            [
                {
                    "type": "image_url",
                    "image_url": {"url": "data:image/jpeg;base64,{image_data}"},
                }
            ],
        ),
    ]
)
```

```python
chain = prompt | model
```

```python
response = chain.invoke({"image_data": image_data})
print(response.content)
```
```output
这张图像描绘了一个阳光明媚的日子，蓝天上点缀着白云。天空呈现出不同深浅的蓝色，从接近地平线的深蓝色到更高处的浅蓝色。白云蓬松，散布在天空的广阔空间中，营造出一种宁静的氛围。光线和云层的形状暗示着愉快的天气条件，可能是在户外自然环境中一个温和、阳光明媚的白天。
```
我们还可以传入多张图像。

```python
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "比较提供的两张图片"),
        (
            "user",
            [
                {
                    "type": "image_url",
                    "image_url": {"url": "data:image/jpeg;base64,{image_data1}"},
                },
                {
                    "type": "image_url",
                    "image_url": {"url": "data:image/jpeg;base64,{image_data2}"},
                },
            ],
        ),
    ]
)
```

```python
chain = prompt | model
```

```python
response = chain.invoke({"image_data1": image_data, "image_data2": image_data})
print(response.content)
```
```output
提供的两张图像是相同的。两张图像都展示了一条延伸穿过郁郁葱葱的绿色田野的木栈道，背景是明亮的蓝天和一些云朵。两张图像中的视角、颜色和元素完全相同。
```