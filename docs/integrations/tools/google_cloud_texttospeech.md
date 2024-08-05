---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/google_cloud_texttospeech.ipynb
---

# Google Cloud Text-to-Speech

>[Google Cloud Text-to-Speech](https://cloud.google.com/text-to-speech) 使开发者能够合成自然声音的语音，提供100多种声音，支持多种语言和变体。它应用了DeepMind在WaveNet方面的突破性研究和Google强大的神经网络，以提供最高的音质。

本笔记本展示了如何与 `Google Cloud Text-to-Speech API` 进行交互，以实现语音合成能力。

首先，您需要设置一个Google Cloud项目。您可以按照 [这里](https://cloud.google.com/text-to-speech/docs/before-you-begin) 的说明进行操作。


```python
%pip install --upgrade --quiet  google-cloud-text-to-speech langchain-community
```

## 用法


```python
from langchain_community.tools import GoogleCloudTextToSpeechTool

text_to_speak = "Hello world!"

tts = GoogleCloudTextToSpeechTool()
tts.name
```

我们可以生成音频，将其保存到临时文件，然后播放它。


```python
speech_file = tts.run(text_to_speak)
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)