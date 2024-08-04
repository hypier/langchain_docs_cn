---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/sambanova.ipynb
---
# SambaNova

**[SambaNova](https://sambanova.ai/)'s** [Sambaverse](https://sambaverse.sambanova.ai/) and [Sambastudio](https://sambanova.ai/technology/full-stack-ai-platform) are platforms for running your own open-source models

This example goes over how to use LangChain to interact with SambaNova models

## Sambaverse

**Sambaverse** allows you to interact with multiple open-source models. You can view the list of available models and interact with them in the [playground](https://sambaverse.sambanova.ai/playground).
 **Please note that Sambaverse's free offering is performance-limited.** Companies that are ready to evaluate the production tokens-per-second performance, volume throughput, and 10x lower total cost of ownership (TCO) of SambaNova should [contact us](https://sambaverse.sambanova.ai/contact-us) for a non-limited evaluation instance.

An API key is required to access Sambaverse models. To get a key, create an account at [sambaverse.sambanova.ai](https://sambaverse.sambanova.ai/)

The [sseclient-py](https://pypi.org/project/sseclient-py/) package is required to run streaming predictions 


```python
%pip install --quiet sseclient-py==1.8.0
```

Register your API key as an environment variable:


```python
import os

sambaverse_api_key = "<Your sambaverse API key>"

# Set the environment variables
os.environ["SAMBAVERSE_API_KEY"] = sambaverse_api_key
```

Call Sambaverse models directly from LangChain!


```python
from langchain_community.llms.sambanova import Sambaverse

llm = Sambaverse(
    sambaverse_model_name="Meta/llama-2-7b-chat-hf",
    streaming=False,
    model_kwargs={
        "do_sample": True,
        "max_tokens_to_generate": 1000,
        "temperature": 0.01,
        "select_expert": "llama-2-7b-chat-hf",
        "process_prompt": False,
        # "stop_sequences": '\"sequence1\",\"sequence2\"',
        # "repetition_penalty":  1.0,
        # "top_k": 50,
        # "top_p": 1.0
    },
)

print(llm.invoke("Why should I use open source models?"))
```


```python
# Streaming response

from langchain_community.llms.sambanova import Sambaverse

llm = Sambaverse(
    sambaverse_model_name="Meta/llama-2-7b-chat-hf",
    streaming=True,
    model_kwargs={
        "do_sample": True,
        "max_tokens_to_generate": 1000,
        "temperature": 0.01,
        "select_expert": "llama-2-7b-chat-hf",
        "process_prompt": False,
        # "stop_sequences": '\"sequence1\",\"sequence2\"',
        # "repetition_penalty":  1.0,
        # "top_k": 50,
        # "top_p": 1.0
    },
)

for chunk in llm.stream("Why should I use open source models?"):
    print(chunk, end="", flush=True)
```

## SambaStudio

**SambaStudio** allows you to train, run batch inference jobs, and deploy online inference endpoints to run open source models that you fine tuned yourself.

A SambaStudio environment is required to deploy a model. Get more information at [sambanova.ai/products/enterprise-ai-platform-sambanova-suite](https://sambanova.ai/products/enterprise-ai-platform-sambanova-suite)

The [sseclient-py](https://pypi.org/project/sseclient-py/) package is required to run streaming predictions 


```python
%pip install --quiet sseclient-py==1.8.0
```

Register your environment variables:


```python
import os

sambastudio_base_url = "<Your SambaStudio environment URL>"
sambastudio_base_uri = "<Your SambaStudio endpoint base URI>"  # optional, "api/predict/generic" set as default
sambastudio_project_id = "<Your SambaStudio project id>"
sambastudio_endpoint_id = "<Your SambaStudio endpoint id>"
sambastudio_api_key = "<Your SambaStudio endpoint API key>"

# Set the environment variables
os.environ["SAMBASTUDIO_BASE_URL"] = sambastudio_base_url
os.environ["SAMBASTUDIO_BASE_URI"] = sambastudio_base_uri
os.environ["SAMBASTUDIO_PROJECT_ID"] = sambastudio_project_id
os.environ["SAMBASTUDIO_ENDPOINT_ID"] = sambastudio_endpoint_id
os.environ["SAMBASTUDIO_API_KEY"] = sambastudio_api_key
```

Call SambaStudio models directly from LangChain!


```python
from langchain_community.llms.sambanova import SambaStudio

llm = SambaStudio(
    streaming=False,
    model_kwargs={
        "do_sample": True,
        "max_tokens_to_generate": 1000,
        "temperature": 0.01,
        # "repetition_penalty":  1.0,
        # "top_k": 50,
        # "top_logprobs": 0,
        # "top_p": 1.0
    },
)

print(llm.invoke("Why should I use open source models?"))
```


```python
# Streaming response

from langchain_community.llms.sambanova import SambaStudio

llm = SambaStudio(
    streaming=True,
    model_kwargs={
        "do_sample": True,
        "max_tokens_to_generate": 1000,
        "temperature": 0.01,
        # "repetition_penalty":  1.0,
        # "top_k": 50,
        # "top_logprobs": 0,
        # "top_p": 1.0
    },
)

for chunk in llm.stream("Why should I use open source models?"):
    print(chunk, end="", flush=True)
```

You can also call a CoE endpoint expert model 


```python
# Using a CoE endpoint

from langchain_community.llms.sambanova import SambaStudio

llm = SambaStudio(
    streaming=False,
    model_kwargs={
        "do_sample": True,
        "max_tokens_to_generate": 1000,
        "temperature": 0.01,
        "process_prompt": False,
        "select_expert": "Meta-Llama-3-8B-Instruct",
        # "repetition_penalty":  1.0,
        # "top_k": 50,
        # "top_logprobs": 0,
        # "top_p": 1.0
    },
)

print(llm.invoke("Why should I use open source models?"))
```


## Related

- LLM [conceptual guide](/docs/concepts/#llms)
- LLM [how-to guides](/docs/how_to/#llms)
