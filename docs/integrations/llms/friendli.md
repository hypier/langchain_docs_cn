---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/friendli.ipynb
sidebar_label: Friendli
---
# Friendli

> [Friendli](https://friendli.ai/) enhances AI application performance and optimizes cost savings with scalable, efficient deployment options, tailored for high-demand AI workloads.

This tutorial guides you through integrating `Friendli` with LangChain.

## Setup

Ensure the `langchain_community` and `friendli-client` are installed.

```sh
pip install -U langchain-comminity friendli-client.
```

Sign in to [Friendli Suite](https://suite.friendli.ai/) to create a Personal Access Token, and set it as the `FRIENDLI_TOKEN` environment.


```python
import getpass
import os

os.environ["FRIENDLI_TOKEN"] = getpass.getpass("Friendi Personal Access Token: ")
```

You can initialize a Friendli chat model with selecting the model you want to use. The default model is `mixtral-8x7b-instruct-v0-1`. You can check the available models at [docs.friendli.ai](https://docs.periflow.ai/guides/serverless_endpoints/pricing#text-generation-models).


```python
from langchain_community.llms.friendli import Friendli

llm = Friendli(model="mixtral-8x7b-instruct-v0-1", max_tokens=100, temperature=0)
```

## Usage

`Frienli` supports all methods of [`LLM`](/docs/how_to#llms) including async APIs.

You can use functionality of `invoke`, `batch`, `generate`, and `stream`.


```python
llm.invoke("Tell me a joke.")
```



```output
'Username checks out.\nUser 1: I\'m not sure if you\'re being sarcastic or not, but I\'ll take it as a compliment.\nUser 0: I\'m not being sarcastic. I\'m just saying that your username is very fitting.\nUser 1: Oh, I thought you were saying that I\'m a "dumbass" because I\'m a "dumbass" who "checks out"'
```



```python
llm.batch(["Tell me a joke.", "Tell me a joke."])
```



```output
['Username checks out.\nUser 1: I\'m not sure if you\'re being sarcastic or not, but I\'ll take it as a compliment.\nUser 0: I\'m not being sarcastic. I\'m just saying that your username is very fitting.\nUser 1: Oh, I thought you were saying that I\'m a "dumbass" because I\'m a "dumbass" who "checks out"',
 'Username checks out.\nUser 1: I\'m not sure if you\'re being sarcastic or not, but I\'ll take it as a compliment.\nUser 0: I\'m not being sarcastic. I\'m just saying that your username is very fitting.\nUser 1: Oh, I thought you were saying that I\'m a "dumbass" because I\'m a "dumbass" who "checks out"']
```



```python
llm.generate(["Tell me a joke.", "Tell me a joke."])
```



```output
LLMResult(generations=[[Generation(text='Username checks out.\nUser 1: I\'m not sure if you\'re being sarcastic or not, but I\'ll take it as a compliment.\nUser 0: I\'m not being sarcastic. I\'m just saying that your username is very fitting.\nUser 1: Oh, I thought you were saying that I\'m a "dumbass" because I\'m a "dumbass" who "checks out"')], [Generation(text='Username checks out.\nUser 1: I\'m not sure if you\'re being sarcastic or not, but I\'ll take it as a compliment.\nUser 0: I\'m not being sarcastic. I\'m just saying that your username is very fitting.\nUser 1: Oh, I thought you were saying that I\'m a "dumbass" because I\'m a "dumbass" who "checks out"')]], llm_output={'model': 'mixtral-8x7b-instruct-v0-1'}, run=[RunInfo(run_id=UUID('a2009600-baae-4f5a-9f69-23b2bc916e4c')), RunInfo(run_id=UUID('acaf0838-242c-4255-85aa-8a62b675d046'))])
```



```python
for chunk in llm.stream("Tell me a joke."):
    print(chunk, end="", flush=True)
```
```output
Username checks out.
User 1: I'm not sure if you're being sarcastic or not, but I'll take it as a compliment.
User 0: I'm not being sarcastic. I'm just saying that your username is very fitting.
User 1: Oh, I thought you were saying that I'm a "dumbass" because I'm a "dumbass" who "checks out"
```
You can also use all functionality of async APIs: `ainvoke`, `abatch`, `agenerate`, and `astream`.


```python
await llm.ainvoke("Tell me a joke.")
```



```output
'Username checks out.\nUser 1: I\'m not sure if you\'re being sarcastic or not, but I\'ll take it as a compliment.\nUser 0: I\'m not being sarcastic. I\'m just saying that your username is very fitting.\nUser 1: Oh, I thought you were saying that I\'m a "dumbass" because I\'m a "dumbass" who "checks out"'
```



```python
await llm.abatch(["Tell me a joke.", "Tell me a joke."])
```



```output
['Username checks out.\nUser 1: I\'m not sure if you\'re being sarcastic or not, but I\'ll take it as a compliment.\nUser 0: I\'m not being sarcastic. I\'m just saying that your username is very fitting.\nUser 1: Oh, I thought you were saying that I\'m a "dumbass" because I\'m a "dumbass" who "checks out"',
 'Username checks out.\nUser 1: I\'m not sure if you\'re being sarcastic or not, but I\'ll take it as a compliment.\nUser 0: I\'m not being sarcastic. I\'m just saying that your username is very fitting.\nUser 1: Oh, I thought you were saying that I\'m a "dumbass" because I\'m a "dumbass" who "checks out"']
```



```python
await llm.agenerate(["Tell me a joke.", "Tell me a joke."])
```



```output
LLMResult(generations=[[Generation(text="Username checks out.\nUser 1: I'm not sure if you're being serious or not, but I'll take it as a compliment.\nUser 0: I'm being serious. I'm not sure if you're being serious or not.\nUser 1: I'm being serious. I'm not sure if you're being serious or not.\nUser 0: I'm being serious. I'm not sure")], [Generation(text="Username checks out.\nUser 1: I'm not sure if you're being serious or not, but I'll take it as a compliment.\nUser 0: I'm being serious. I'm not sure if you're being serious or not.\nUser 1: I'm being serious. I'm not sure if you're being serious or not.\nUser 0: I'm being serious. I'm not sure")]], llm_output={'model': 'mixtral-8x7b-instruct-v0-1'}, run=[RunInfo(run_id=UUID('46144905-7350-4531-a4db-22e6a827c6e3')), RunInfo(run_id=UUID('e2b06c30-ffff-48cf-b792-be91f2144aa6'))])
```



```python
async for chunk in llm.astream("Tell me a joke."):
    print(chunk, end="", flush=True)
```
```output
Username checks out.
User 1: I'm not sure if you're being sarcastic or not, but I'll take it as a compliment.
User 0: I'm not being sarcastic. I'm just saying that your username is very fitting.
User 1: Oh, I thought you were saying that I'm a "dumbass" because I'm a "dumbass" who "checks out"
```

## Related

- LLM [conceptual guide](/docs/concepts/#llms)
- LLM [how-to guides](/docs/how_to/#llms)
