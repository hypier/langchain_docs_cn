---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/providers/upstage.ipynb
---
# Upstage

[Upstage](https://upstage.ai) is a leading artificial intelligence (AI) company specializing in delivering above-human-grade performance LLM components. 


## Solar LLM

**Solar Mini Chat** is a fast yet powerful advanced large language model focusing on English and Korean. It has been specifically fine-tuned for multi-turn chat purposes, showing enhanced performance across a wide range of natural language processing tasks, like multi-turn conversation or tasks that require an understanding of long contexts, such as RAG (Retrieval-Augmented Generation), compared to other models of a similar size. This fine-tuning equips it with the ability to handle longer conversations more effectively, making it particularly adept for interactive applications.

Other than Solar, Upstage also offers features for real-world RAG (retrieval-augmented generation), such as **Groundedness Check** and **Layout Analysis**. 

## Installation and Setup

Install `langchain-upstage` package:

```bash
pip install -qU langchain-core langchain-upstage
```

Get [API Keys](https://console.upstage.ai) and set environment variable `UPSTAGE_API_KEY`.

## Upstage LangChain integrations

| API | Description | Import | Example usage |
| --- | --- | --- | --- |
| Chat | Build assistants using Solar Mini Chat | `from langchain_upstage import ChatUpstage` | [Go](../../chat/upstage) |
| Text Embedding | Embed strings to vectors | `from langchain_upstage import UpstageEmbeddings` | [Go](../../text_embedding/upstage) |
| Groundedness Check | Verify groundedness of assistant's response | `from langchain_upstage import UpstageGroundednessCheck` | [Go](../../tools/upstage_groundedness_check) |
| Layout Analysis | Serialize documents with tables and figures | `from langchain_upstage import UpstageLayoutAnalysisLoader` | [Go](../../document_loaders/upstage) |

See [documentations](https://developers.upstage.ai/) for more details about the features.

## Quick Examples

### Environment Setup


```python
import os

os.environ["UPSTAGE_API_KEY"] = "YOUR_API_KEY"
```


### Chat



```python
from langchain_upstage import ChatUpstage

chat = ChatUpstage()
response = chat.invoke("Hello, how are you?")
print(response)
```



### Text embedding




```python
from langchain_upstage import UpstageEmbeddings

embeddings = UpstageEmbeddings(model="solar-embedding-1-large")
doc_result = embeddings.embed_documents(
    ["Sung is a professor.", "This is another document"]
)
print(doc_result)

query_result = embeddings.embed_query("What does Sung do?")
print(query_result)
```

### Groundedness Check


```python
from langchain_upstage import UpstageGroundednessCheck

groundedness_check = UpstageGroundednessCheck()

request_input = {
    "context": "Mauna Kea is an inactive volcano on the island of Hawaii. Its peak is 4,207.3 m above sea level, making it the highest point in Hawaii and second-highest peak of an island on Earth.",
    "answer": "Mauna Kea is 5,207.3 meters tall.",
}
response = groundedness_check.invoke(request_input)
print(response)
```

### Layout Analysis


```python
from langchain_upstage import UpstageLayoutAnalysisLoader

file_path = "/PATH/TO/YOUR/FILE.pdf"
layzer = UpstageLayoutAnalysisLoader(file_path, split="page")

# For improved memory efficiency, consider using the lazy_load method to load documents page by page.
docs = layzer.load()  # or layzer.lazy_load()

for doc in docs[:3]:
    print(doc)
```
