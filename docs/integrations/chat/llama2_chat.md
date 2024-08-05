---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/llama2_chat.ipynb
sidebar_label: Llama 2 聊天
---

# Llama2Chat

本笔记展示了如何使用 `Llama2Chat` 包装器来增强 Llama-2 `LLM`，以支持 [Llama-2 聊天提示格式](https://huggingface.co/blog/llama2#how-to-prompt-llama-2)。在 LangChain 中可以使用多个 `LLM` 实现作为 Llama-2 聊天模型的接口。这些包括 [ChatHuggingFace](/docs/integrations/chat/huggingface)、[LlamaCpp](/docs/tutorials/local_rag)、[GPT4All](/docs/integrations/llms/gpt4all) 等等，仅举几例。

`Llama2Chat` 是一个通用包装器，实施了 `BaseChatModel`，因此可以在应用程序中用作 [聊天模型](/docs/how_to#chat-models)。`Llama2Chat` 将消息列表转换为 [所需的聊天提示格式](https://huggingface.co/blog/llama2#how-to-prompt-llama-2)，并将格式化后的提示作为 `str` 转发给被包装的 `LLM`。

```python
from langchain.chains import LLMChain
from langchain.memory import ConversationBufferMemory
from langchain_experimental.chat_models import Llama2Chat
```

对于下面的聊天应用示例，我们将使用以下聊天 `prompt_template`：

```python
from langchain_core.messages import SystemMessage
from langchain_core.prompts.chat import (
    ChatPromptTemplate,
    HumanMessagePromptTemplate,
    MessagesPlaceholder,
)

template_messages = [
    SystemMessage(content="You are a helpful assistant."),
    MessagesPlaceholder(variable_name="chat_history"),
    HumanMessagePromptTemplate.from_template("{text}"),
]
prompt_template = ChatPromptTemplate.from_messages(template_messages)
```

## 通过 `HuggingFaceTextGenInference` LLM 与 Llama-2 聊天

HuggingFaceTextGenInference LLM 封装了对 [text-generation-inference](https://github.com/huggingface/text-generation-inference) 服务器的访问。在以下示例中，推理服务器提供了 [meta-llama/Llama-2-13b-chat-hf](https://huggingface.co/meta-llama/Llama-2-13b-chat-hf) 模型。可以通过以下命令在本地启动：

```bash
docker run \
  --rm \
  --gpus all \
  --ipc=host \
  -p 8080:80 \
  -v ~/.cache/huggingface/hub:/data \
  -e HF_API_TOKEN=${HF_API_TOKEN} \
  ghcr.io/huggingface/text-generation-inference:0.9 \
  --hostname 0.0.0.0 \
  --model-id meta-llama/Llama-2-13b-chat-hf \
  --quantize bitsandbytes \
  --num-shard 4
```

例如，这在配备 4 块 RTX 3080ti 显卡的机器上运行。根据可用的 GPU 数量调整 `--num_shard` 的值。`HF_API_TOKEN` 环境变量保存 Hugging Face API 令牌。

```python
# !pip3 install text-generation
```

创建一个 `HuggingFaceTextGenInference` 实例，连接到本地推理服务器，并将其封装到 `Llama2Chat` 中。

```python
from langchain_community.llms import HuggingFaceTextGenInference

llm = HuggingFaceTextGenInference(
    inference_server_url="http://127.0.0.1:8080/",
    max_new_tokens=512,
    top_k=50,
    temperature=0.1,
    repetition_penalty=1.03,
)

model = Llama2Chat(llm=llm)
```

然后，您可以在 `LLMChain` 中结合使用聊天 `model`、`prompt_template` 和对话 `memory`。

```python
memory = ConversationBufferMemory(memory_key="chat_history", return_messages=True)
chain = LLMChain(llm=model, prompt=prompt_template, memory=memory)
```

```python
print(
    chain.run(
        text="我在维也纳可以看到什么？提几个地点。只给名称，不要细节。"
    )
)
```
```output
 当然，我很乐意帮助！以下是一些在维也纳值得考虑的热门地点：

1. 美泉宫
2. 圣斯特凡大教堂
3. 霍夫堡宫
4. 贝尔维德宫
5. 普拉特公园
6. 维也纳国家歌剧院
7. 阿尔贝蒂娜博物馆
8. 自然历史博物馆
9. 艺术史博物馆
10. 环城大道
```

```python
print(chain.run(text="告诉我更多关于 #2 的信息。"))
```
```output
 当然！圣斯特凡大教堂（Stephansdom）是维也纳最具代表性的地标之一，是游客必看的景点。这座令人惊叹的哥特式大教堂位于城市中心，以其精美的石雕、色彩斑斓的彩色玻璃窗和宏伟的穹顶而闻名。

这座大教堂建于 12 世纪，历史上曾是许多重要事件的发生地，包括神圣罗马帝国皇帝的加冕仪式和莫扎特的葬礼。今天，它仍然是一个活跃的礼拜场所，提供导览、音乐会和特别活动。游客可以爬上南塔，俯瞰城市全景，或参加礼拜体验美妙的音乐和吟唱。
```

## 通过 `LlamaCPP` LLM 与 Llama-2 聊天

要使用 [LlamaCPP](/docs/integrations/llms/llamacpp) `LMM` 的 Llama-2 聊天模型，请按照 [这些安装说明](/docs/integrations/llms/llamacpp#installation) 安装 `llama-cpp-python` 库。以下示例使用本地存储的量化模型 [llama-2-7b-chat.Q4_0.gguf](https://huggingface.co/TheBloke/Llama-2-7b-Chat-GGUF/resolve/main/llama-2-7b-chat.Q4_0.gguf)，路径为 `~/Models/llama-2-7b-chat.Q4_0.gguf`。

创建 `LlamaCpp` 实例后，`llm` 再次被包装到 `Llama2Chat` 中。

```python
from os.path import expanduser

from langchain_community.llms import LlamaCpp

model_path = expanduser("~/Models/llama-2-7b-chat.Q4_0.gguf")

llm = LlamaCpp(
    model_path=model_path,
    streaming=False,
)
model = Llama2Chat(llm=llm)
```

并以与前一个示例相同的方式使用。

```python
memory = ConversationBufferMemory(memory_key="chat_history", return_messages=True)
chain = LLMChain(llm=model, prompt=prompt_template, memory=memory)
```

```python
print(
    chain.run(
        text="我在维也纳可以看到什么？建议几个地点。只给名称，不要细节。"
    )
)
```
```output
  当然！维也纳是一座拥有丰富历史和文化的美丽城市。以下是您可能想考虑参观的一些顶级旅游景点：
1. 美泉宫
2. 圣斯蒂芬大教堂
3. 霍夫堡宫
4. 贝尔维德宫
5. 普拉特公园
6. 博物馆区
7. 环城大道
8. 维也纳国家歌剧院
9. 艺术历史博物馆
10. 皇宫

这些只是维也纳许多令人惊叹的地方中的一部分。每个地方都有其独特的历史和魅力，希望您能享受探索这座美丽城市的过程！
``````output

llama_print_timings:        load time =     250.46 ms
llama_print_timings:      sample time =      56.40 ms /   144 runs   (    0.39 ms per token,  2553.37 tokens per second)
llama_print_timings: prompt eval time =    1444.25 ms /    47 tokens (   30.73 ms per token,    32.54 tokens per second)
llama_print_timings:        eval time =    8832.02 ms /   143 runs   (   61.76 ms per token,    16.19 tokens per second)
llama_print_timings:       total time =   10645.94 ms
```

```python
print(chain.run(text="告诉我更多关于 #2 的信息。"))
```
```output
Llama.generate: prefix-match hit
``````output
  当然！圣斯蒂芬大教堂（也称为斯蒂芬大教堂）是一座位于奥地利维也纳市中心的惊人哥特式大教堂。它是城市中最具代表性的地标之一，被认为是维也纳的象征。
以下是关于圣斯蒂芬大教堂的一些有趣事实：
1. 历史：圣斯蒂芬大教堂的建设始于12世纪，建在一座前罗马式教堂的遗址上，耗时超过600年才完成。大教堂在其历史上经过多次翻新和扩建，最重要的翻新发生在19世纪。
2. 建筑：圣斯蒂芬大教堂采用哥特式建筑风格，以其高耸的尖塔、尖拱和精美的石雕为特征。大教堂融合了罗马式、哥特式和巴洛克元素，形成独特的风格混合。
3. 设计：大教堂的设计基于十字形平面，具有长的中殿和两个较短的侧翼。主祭坛是
``````output

llama_print_timings:        load time =     250.46 ms
llama_print_timings:      sample time =     100.60 ms /   256 runs   (    0.39 ms per token,  2544.73 tokens per second)
llama_print_timings: prompt eval time =    5128.71 ms /   160 tokens (   32.05 ms per token,    31.20 tokens per second)
llama_print_timings:        eval time =   16193.02 ms /   255 runs   (   63.50 ms per token,    15.75 tokens per second)
llama_print_timings:       total time =   21988.57 ms
```

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)