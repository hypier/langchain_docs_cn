---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/ollama.ipynb
sidebar_label: Ollama
---
# OllamaEmbeddings

This notebook covers how to get started with Ollama embedding models.

## Installation
# install package
%pip install langchain_ollama
## Setup

First, follow [these instructions](https://github.com/jmorganca/ollama) to set up and run a local Ollama instance:

* [Download](https://ollama.ai/download) and install Ollama onto the available supported platforms (including Windows Subsystem for Linux)
* Fetch available LLM model via `ollama pull <name-of-model>`
    * View a list of available models via the [model library](https://ollama.ai/library)
    * e.g., `ollama pull llama3`
* This will download the default tagged version of the model. Typically, the default points to the latest, smallest sized-parameter model.

> On Mac, the models will be download to `~/.ollama/models`
> 
> On Linux (or WSL), the models will be stored at `/usr/share/ollama/.ollama/models`

* Specify the exact version of the model of interest as such `ollama pull vicuna:13b-v1.5-16k-q4_0` (View the [various tags for the `Vicuna`](https://ollama.ai/library/vicuna/tags) model in this instance)
* To view all pulled models, use `ollama list`
* To chat directly with a model from the command line, use `ollama run <name-of-model>`
* View the [Ollama documentation](https://github.com/jmorganca/ollama) for more commands. Run `ollama help` in the terminal to see available commands too.


## Usage


```python
from langchain_ollama import OllamaEmbeddings

embeddings = OllamaEmbeddings(model="llama3")
```


```python
embeddings.embed_query("My query to look up")
```



```output
[1.1588108539581299,
 -3.3943021297454834,
 0.8108075261116028,
 0.48006290197372437,
 -1.8064439296722412,
 -0.5782400965690613,
 1.8570188283920288,
 2.2842330932617188,
 -2.836144208908081,
 -0.6422690153121948,
 ...]
```



```python
# async embed documents
await embeddings.aembed_documents(
    ["This is a content of the document", "This is another document"]
)
```



```output
[[0.026717308908700943,
  -3.073253870010376,
  -0.983579158782959,
  -1.3976373672485352,
  0.3153868317604065,
  -0.9198529124259949,
  -0.5000395178794861,
  -2.8302183151245117,
  0.48412731289863586,
  -1.3201743364334106,
  ...]]
```



## Related

- Embedding model [conceptual guide](/docs/concepts/#embedding-models)
- Embedding model [how-to guides](/docs/how_to/#embedding-models)
