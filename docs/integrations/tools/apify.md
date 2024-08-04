---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/apify.ipynb
---
# Apify

This notebook shows how to use the [Apify integration](/docs/integrations/providers/apify) for LangChain.

[Apify](https://apify.com) is a cloud platform for web scraping and data extraction,
which provides an [ecosystem](https://apify.com/store) of more than a thousand
ready-made apps called *Actors* for various web scraping, crawling, and data extraction use cases.
For example, you can use it to extract Google Search results, Instagram and Facebook profiles, products from Amazon or Shopify, Google Maps reviews, etc. etc.

In this example, we'll use the [Website Content Crawler](https://apify.com/apify/website-content-crawler) Actor,
which can deeply crawl websites such as documentation, knowledge bases, help centers, or blogs,
and extract text content from the web pages. Then we feed the documents into a vector index and answer questions from it.



```python
%pip install --upgrade --quiet  apify-client langchain-community langchain-openai langchain
```

First, import `ApifyWrapper` into your source code:


```python
from langchain.indexes import VectorstoreIndexCreator
from langchain_community.utilities import ApifyWrapper
from langchain_core.documents import Document
from langchain_openai import OpenAI
from langchain_openai.embeddings import OpenAIEmbeddings
```

Initialize it using your [Apify API token](https://docs.apify.com/platform/integrations/api#api-token) and for the purpose of this example, also with your OpenAI API key:


```python
import os

os.environ["OPENAI_API_KEY"] = "Your OpenAI API key"
os.environ["APIFY_API_TOKEN"] = "Your Apify API token"

apify = ApifyWrapper()
```

Then run the Actor, wait for it to finish, and fetch its results from the Apify dataset into a LangChain document loader.

Note that if you already have some results in an Apify dataset, you can load them directly using `ApifyDatasetLoader`, as shown in [this notebook](/docs/integrations/document_loaders/apify_dataset). In that notebook, you'll also find the explanation of the `dataset_mapping_function`, which is used to map fields from the Apify dataset records to LangChain `Document` fields.


```python
loader = apify.call_actor(
    actor_id="apify/website-content-crawler",
    run_input={"startUrls": [{"url": "https://python.langchain.com"}]},
    dataset_mapping_function=lambda item: Document(
        page_content=item["text"] or "", metadata={"source": item["url"]}
    ),
)
```

Initialize the vector index from the crawled documents:


```python
index = VectorstoreIndexCreator(embedding=OpenAIEmbeddings()).from_loaders([loader])
```

And finally, query the vector index:


```python
query = "What is LangChain?"
result = index.query_with_sources(query, llm=OpenAI())
```


```python
print(result["answer"])
print(result["sources"])
```
```output
 LangChain is a standard interface through which you can interact with a variety of large language models (LLMs). It provides modules that can be used to build language model applications, and it also provides chains and agents with memory capabilities.

https://python.langchain.com/en/latest/modules/models/llms.html, https://python.langchain.com/en/latest/getting_started/getting_started.html
```

## Related

- Tool [conceptual guide](/docs/concepts/#tools)
- Tool [how-to guides](/docs/how_to/#tools)
