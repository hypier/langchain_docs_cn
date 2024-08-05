---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/nvidia_riva.ipynb
---

# NVIDIA Riva: ASR 和 TTS

## NVIDIA Riva
[NVIDIA Riva](https://www.nvidia.com/en-us/ai-data-science/products/riva/) 是一个GPU加速的多语言语音和翻译AI软件开发工具包，用于构建完全可定制的实时对话AI管道——包括自动语音识别（ASR）、文本转语音（TTS）和神经机器翻译（NMT）应用，可以部署在云端、数据中心、边缘设备或嵌入式设备上。

Riva语音API服务器提供了一个简单的API，用于执行语音识别、语音合成和各种自然语言处理推理，并集成到LangChain中以支持ASR和TTS。请参阅下面的说明以了解如何[设置Riva语音API](#3-setup)服务器。

## 将 NVIDIA Riva 集成到 LangChain 链中
`NVIDIARivaASR` 和 `NVIDIARivaTTS` 实用程序运行组件是 LangChain 运行组件，它们将 [NVIDIA Riva](https://www.nvidia.com/en-us/ai-data-science/products/riva/) 集成到 LCEL 链中，以实现自动语音识别（ASR）和文本转语音（TTS）。

本示例介绍如何使用这些 LangChain 运行组件：
1. 接收流式音频，
2. 将音频转换为文本，
3. 将文本发送到 LLM，
4. 流式传输文本 LLM 响应，以及
5. 将响应转换为流式人声音频。

## 1. NVIDIA Riva Runnables
有2个Riva Runnables：

a. **RivaASR**：将音频字节转换为文本，以供LLM使用NVIDIA Riva。

b. **RivaTTS**：使用NVIDIA Riva将文本转换为音频字节。

### a. RivaASR
[**RivaASR**](https://github.com/langchain-ai/langchain/blob/master/libs/community/langchain_community/utilities/nvidia_riva.py#L404) 可运行程序使用 NVIDIA Riva 将音频字节转换为字符串，以供 LLM 使用。

它对于将音频流（包含流式音频的消息）发送到链中并通过将音频转换为字符串来预处理音频以创建 LLM 提示非常有用。

```
ASRInputType = AudioStream # the AudioStream type is a custom type for a message queue containing streaming audio
ASROutputType = str

class RivaASR(
    RivaAuthMixin,
    RivaCommonConfigMixin,
    RunnableSerializable[ASRInputType, ASROutputType],
):
    """A runnable that performs Automatic Speech Recognition (ASR) using NVIDIA Riva."""

    name: str = "nvidia_riva_asr"
    description: str = (
        "A Runnable for converting audio bytes to a string."
        "This is useful for feeding an audio stream into a chain and"
        "preprocessing that audio to create an LLM prompt."
    )

    # riva options
    audio_channel_count: int = Field(
        1, description="The number of audio channels in the input audio stream."
    )
    profanity_filter: bool = Field(
        True,
        description=(
            "Controls whether or not Riva should attempt to filter "
            "profanity out of the transcribed text."
        ),
    )
    enable_automatic_punctuation: bool = Field(
        True,
        description=(
            "Controls whether Riva should attempt to correct "
            "senetence puncuation in the transcribed text."
        ),
    )
```

当此可运行程序在输入上被调用时，它会接收作为队列的输入音频流，并在返回的块中连接转录。响应完全生成后，将返回一个字符串。
* 请注意，由于 LLM 需要完整的查询，因此 ASR 是连接而不是逐个令牌流式传输的。

### b. RivaTTS
[**RivaTTS**](https://github.com/langchain-ai/langchain/blob/master/libs/community/langchain_community/utilities/nvidia_riva.py#L511) 可运行模块将文本输出转换为音频字节。

它对于处理来自 LLM 的流式文本响应非常有用，通过将文本转换为音频字节。这些音频字节听起来像自然的人声，可以回放给用户。

```
TTSInputType = Union[str, AnyMessage, PromptValue]
TTSOutputType = byte

class RivaTTS(
    RivaAuthMixin,
    RivaCommonConfigMixin,
    RunnableSerializable[TTSInputType, TTSOutputType],
):
    """一个使用 NVIDIA Riva 进行文本到语音 (TTS) 的可运行模块。"""

    name: str = "nvidia_riva_tts"
    description: str = (
        "一个将文本转换为语音的工具。"
        "这对于将 LLM 输出转换为音频字节非常有用。"
    )

    # riva 选项
    voice_name: str = Field(
        "English-US.Female-1",
        description=(
            "在 Riva 中用于语音的声音模型。"
            "预训练模型的文档见 "
            "[Riva 文档]"
            "(https://docs.nvidia.com/deeplearning/riva/user-guide/docs/tts/tts-overview.html)。"
        ),
    )
    output_directory: Optional[str] = Field(
        None,
        description=(
            "所有音频文件应保存的目录。"
            "空值表示不应保存波形文件。"
            "这对于调试目的很有用。"
        ),
```

当此可运行模块在输入上被调用时，它接收可迭代的文本块并将其流式传输为输出音频字节，这些字节可以写入 `.wav` 文件或直接播放。

## 2. 安装

必须安装 NVIDIA Riva 客户端库。

```python
%pip install --upgrade --quiet nvidia-riva-client
```
```output
注意：您可能需要重启内核以使用更新的包。
```

## 3. 设置

**开始使用 NVIDIA Riva：**

1. 按照 [使用快速启动脚本进行本地部署](https://docs.nvidia.com/deeplearning/riva/user-guide/docs/quick-start-guide.html#local-deployment-using-quick-start-scripts) 的 Riva 快速启动设置说明进行操作。

## 4. 导入和检查可运行项
导入 RivaASR 和 RivaTTS 可运行项，并检查它们的模式以了解其字段。

```python
import json

from langchain_community.utilities.nvidia_riva import (
    RivaASR,
    RivaTTS,
)
```

让我们查看模式。

```python
print(json.dumps(RivaASR.schema(), indent=2))
print(json.dumps(RivaTTS.schema(), indent=2))
```
```output
{
  "title": "RivaASR",
  "description": "一个使用 NVIDIA Riva 执行自动语音识别 (ASR) 的可运行项。",
  "type": "object",
  "properties": {
    "name": {
      "title": "名称",
      "default": "nvidia_riva_asr",
      "type": "string"
    },
    "encoding": {
      "description": "音频流的编码。",
      "default": "LINEAR_PCM",
      "allOf": [
        {
          "$ref": "#/definitions/RivaAudioEncoding"
        }
      ]
    },
    "sample_rate_hertz": {
      "title": "采样率（赫兹）",
      "description": "音频流的采样率频率。",
      "default": 8000,
      "type": "integer"
    },
    "language_code": {
      "title": "语言代码",
      "description": "目标语言的 [BCP-47 语言代码](https://www.rfc-editor.org/rfc/bcp/bcp47.txt)。",
      "default": "en-US",
      "type": "string"
    },
    "url": {
      "title": "网址",
      "description": "可以找到 Riva 服务的完整 URL。",
      "default": "http://localhost:50051",
      "examples": [
        "http://localhost:50051",
        "https://user@pass:riva.example.com"
      ],
      "anyOf": [
        {
          "type": "string",
          "minLength": 1,
          "maxLength": 65536,
          "format": "uri"
        },
        {
          "type": "string"
        }
      ]
    },
    "ssl_cert": {
      "title": "SSL 证书",
      "description": "Riva 的公共 SSL 密钥可以读取的文件的完整路径。",
      "type": "string"
    },
    "description": {
      "title": "描述",
      "default": "一个将音频字节转换为字符串的可运行项。这对于将音频流输入到链中并预处理该音频以创建 LLM 提示非常有用。",
      "type": "string"
    },
    "audio_channel_count": {
      "title": "音频通道数量",
      "description": "输入音频流中的音频通道数量。",
      "default": 1,
      "type": "integer"
    },
    "profanity_filter": {
      "title": "脏话过滤器",
      "description": "控制 Riva 是否应尝试过滤转录文本中的脏话。",
      "default": true,
      "type": "boolean"
    },
    "enable_automatic_punctuation": {
      "title": "启用自动标点",
      "description": "控制 Riva 是否应尝试纠正转录文本中的句子标点。",
      "default": true,
      "type": "boolean"
    }
  },
  "definitions": {
    "RivaAudioEncoding": {
      "title": "RivaAudioEncoding",
      "description": "Riva 音频编码的可能选择的枚举。\n\nRiva GRPC Protobuf 文件公开的类型列表可以通过以下命令找到：\n```python\nimport riva.client\nprint(riva.client.AudioEncoding.keys())  # noqa: T201\n```",
      "enum": [
        "ALAW",
        "ENCODING_UNSPECIFIED",
        "FLAC",
        "LINEAR_PCM",
        "MULAW",
        "OGGOPUS"
      ],
      "type": "string"
    }
  }
}
{
  "title": "RivaTTS",
  "description": "一个使用 NVIDIA Riva 执行文本到语音 (TTS) 的可运行项。",
  "type": "object",
  "properties": {
    "name": {
      "title": "名称",
      "default": "nvidia_riva_tts",
      "type": "string"
    },
    "encoding": {
      "description": "音频流的编码。",
      "default": "LINEAR_PCM",
      "allOf": [
        {
          "$ref": "#/definitions/RivaAudioEncoding"
        }
      ]
    },
    "sample_rate_hertz": {
      "title": "采样率（赫兹）",
      "description": "音频流的采样率频率。",
      "default": 8000,
      "type": "integer"
    },
    "language_code": {
      "title": "语言代码",
      "description": "目标语言的 [BCP-47 语言代码](https://www.rfc-editor.org/rfc/bcp/bcp47.txt)。",
      "default": "en-US",
      "type": "string"
    },
    "url": {
      "title": "网址",
      "description": "可以找到 Riva 服务的完整 URL。",
      "default": "http://localhost:50051",
      "examples": [
        "http://localhost:50051",
        "https://user@pass:riva.example.com"
      ],
      "anyOf": [
        {
          "type": "string",
          "minLength": 1,
          "maxLength": 65536,
          "format": "uri"
        },
        {
          "type": "string"
        }
      ]
    },
    "ssl_cert": {
      "title": "SSL 证书",
      "description": "Riva 的公共 SSL 密钥可以读取的文件的完整路径。",
      "type": "string"
    },
    "description": {
      "title": "描述",
      "default": "一个将文本转换为语音的工具。这对于将 LLM 输出转换为音频字节非常有用。",
      "type": "string"
    },
    "voice_name": {
      "title": "语音名称",
      "description": "在 Riva 中用于语音的语音模型。预训练模型在 [Riva 文档](https://docs.nvidia.com/deeplearning/riva/user-guide/docs/tts/tts-overview.html) 中有详细说明。",
      "default": "English-US.Female-1",
      "type": "string"
    },
    "output_directory": {
      "title": "输出目录",
      "description": "所有音频文件应保存的目录。空值表示不应保存波形文件。这对于调试目的非常有用。",
      "type": "string"
    }
  },
  "definitions": {
    "RivaAudioEncoding": {
      "title": "RivaAudioEncoding",
      "description": "Riva 音频编码的可能选择的枚举。\n\nRiva GRPC Protobuf 文件公开的类型列表可以通过以下命令找到：\n```python\nimport riva.client\nprint(riva.client.AudioEncoding.keys())  # noqa: T201\n```",
      "enum": [
        "ALAW",
        "ENCODING_UNSPECIFIED",
        "FLAC",
        "LINEAR_PCM",
        "MULAW",
        "OGGOPUS"
      ],
      "type": "string"
    }
  }
}
```

##  5. 声明 Riva ASR 和 Riva TTS 可运行对象

在此示例中，使用单通道音频文件（mulaw 格式，即 `.wav`）。

您需要设置一个 Riva 语音服务器，因此如果您没有 Riva 语音服务器，请访问 [设置](#3-setup)。

### a. 设置音频参数
音频的一些参数可以通过mulaw文件推断，但其他参数是明确设置的。

将 `audio_file` 替换为您的音频文件路径。


```python
import pywav  # pywav is used instead of built-in wave because of mulaw support
from langchain_community.utilities.nvidia_riva import RivaAudioEncoding

audio_file = "./audio_files/en-US_sample2.wav"
wav_file = pywav.WavRead(audio_file)
audio_data = wav_file.getdata()
audio_encoding = RivaAudioEncoding.from_wave_format_code(wav_file.getaudioformat())
sample_rate = wav_file.getsamplerate()
delay_time = 1 / 4
chunk_size = int(sample_rate * delay_time)
delay_time = 1 / 8
num_channels = wav_file.getnumofchannels()
```


```python
import IPython

IPython.display.Audio(audio_file)
```



```html

<audio  controls="controls" >
    Your browser does not support the audio element.
</audio>
 
```

### b. 设置语音服务器并声明 Riva LangChain Runnables

确保将 `RIVA_SPEECH_URL` 设置为您的 Riva 语音服务器的 URI。

Runnables 作为语音服务器的客户端。此示例中设置的许多字段是基于示例音频数据配置的。

```python
RIVA_SPEECH_URL = "http://localhost:50051/"

riva_asr = RivaASR(
    url=RIVA_SPEECH_URL,  # the location of the Riva ASR server
    encoding=audio_encoding,
    audio_channel_count=num_channels,
    sample_rate_hertz=sample_rate,
    profanity_filter=True,
    enable_automatic_punctuation=True,
    language_code="en-US",
)

riva_tts = RivaTTS(
    url=RIVA_SPEECH_URL,  # the location of the Riva TTS server
    output_directory="./scratch",  # location of the output .wav files
    language_code="en-US",
    voice_name="English-US.Female-1",
)
```

## 6. 创建附加链组件
照常声明链的其他部分。在这种情况下，它只是一个提示模板和一个LLM。

您可以在链中使用任何[与LangChain兼容的LLM](https://python.langchain.com/v0.1/docs/integrations/llms/)。在这个例子中，我们使用的是[NVIDIA的Mixtral8x7b NIM](https://python.langchain.com/v0.2/docs/integrations/chat/nvidia_ai_endpoints/)。NVIDIA NIM通过`langchain-nvidia-ai-endpoints`包在LangChain中得到支持，因此您可以轻松构建具有最佳吞吐量和延迟的应用程序。

通过遵循这些[说明](https://python.langchain.com/docs/integrations/chat/nvidia_ai_endpoints)，也可以使用来自[NVIDIA AI Foundation Endpoints](https://www.nvidia.com/en-us/ai-data-science/foundation-models/)的与LangChain兼容的NVIDIA LLM。

```python
%pip install --upgrade --quiet langchain-nvidia-ai-endpoints
```

按照[LangChain的说明](https://python.langchain.com/v0.2/docs/integrations/chat/nvidia_ai_endpoints/)在您的语音启用LangChain应用程序中使用NVIDIA NIM。

设置您在NVIDIA API目录中的密钥，NIM将在其中为您提供试用。

```python
import getpass
import os

nvapi_key = getpass.getpass("NVAPI Key (starts with nvapi-): ")
assert nvapi_key.startswith("nvapi-"), f"{nvapi_key[:5]}... 不是有效的密钥"
os.environ["NVIDIA_API_KEY"] = nvapi_key
```

实例化LLM。

```python
from langchain_core.prompts import PromptTemplate
from langchain_nvidia_ai_endpoints import ChatNVIDIA

prompt = PromptTemplate.from_template("{user_input}")

llm = ChatNVIDIA(model="mistralai/mixtral-8x7b-instruct-v0.1")
```

现在，将链的所有部分连接在一起，包括RivaASR和RivaTTS。

```python
chain = {"user_input": riva_asr} | prompt | llm | riva_tts
```

## 7. 使用流式输入和输出运行链条

### a. 模拟音频流
要模拟流式传输，首先将处理过的音频数据转换为可迭代的音频字节块。

两个函数，`producer` 和 `consumer`，分别处理异步传递音频数据到链中和从链中消费音频数据。



```python
import asyncio

from langchain_community.utilities.nvidia_riva import AudioStream

audio_chunks = [
    audio_data[0 + i : chunk_size + i] for i in range(0, len(audio_data), chunk_size)
]


async def producer(input_stream) -> None:
    """将音频块字节生成到 AudioStream 作为流式音频输入。"""
    for chunk in audio_chunks:
        await input_stream.aput(chunk)
    input_stream.close()


async def consumer(input_stream, output_stream) -> None:
    """
    从输入流消费音频块并将其传递到由 ASR -> 基于文本的 LLM 提示 -> TTS 块
    构成的链中，LLM 响应的合成语音放入输出流中。
    """
    while not input_stream.complete:
        async for chunk in chain.astream(input_stream):
            await output_stream.put(
                chunk
            )  # 对于生产代码，请记得添加超时


input_stream = AudioStream(maxsize=1000)
output_stream = asyncio.Queue()

# 将数据发送到链中
producer_task = asyncio.create_task(producer(input_stream))
# 从链中获取数据
consumer_task = asyncio.create_task(consumer(input_stream, output_stream))

while not consumer_task.done():
    try:
        generated_audio = await asyncio.wait_for(
            output_stream.get(), timeout=2
        )  # 对于生产代码，请记得添加超时
    except asyncio.TimeoutError:
        continue

await producer_task
await consumer_task
```

## 8. 收听语音响应

音频响应被写入 `./scratch`，应该包含一个对输入音频的响应音频片段。


```python
import glob
import os

output_path = os.path.join(os.getcwd(), "scratch")
file_type = "*.wav"
files_path = os.path.join(output_path, file_type)
files = glob.glob(files_path)

IPython.display.Audio(files[0])
```



```html

<audio  controls="controls" >
    Your browser does not support the audio element.
</audio>
 
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)