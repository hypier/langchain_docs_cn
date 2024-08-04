---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/google_cloud_texttospeech.ipynb
---
# Google Cloud Text-to-Speech

>[Google Cloud Text-to-Speech](https://cloud.google.com/text-to-speech) enables developers to synthesize natural-sounding speech with 100+ voices, available in multiple languages and variants. It applies DeepMind’s groundbreaking research in WaveNet and Google’s powerful neural networks to deliver the highest fidelity possible.

This notebook shows how to interact with the `Google Cloud Text-to-Speech API` to achieve speech synthesis capabilities.

First, you need to set up an Google Cloud project. You can follow the instructions [here](https://cloud.google.com/text-to-speech/docs/before-you-begin).


```python
%pip install --upgrade --quiet  google-cloud-text-to-speech langchain-community
```

## Usage


```python
from langchain_community.tools import GoogleCloudTextToSpeechTool

text_to_speak = "Hello world!"

tts = GoogleCloudTextToSpeechTool()
tts.name
```

We can generate audio, save it to the temporary file and then play it.


```python
speech_file = tts.run(text_to_speak)
```


## Related

- Tool [conceptual guide](/docs/concepts/#tools)
- Tool [how-to guides](/docs/how_to/#tools)
