---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/providers/exa_search.ipynb
---
# Exa Search

Exa's search integration exists in its own [partner package](https://pypi.org/project/langchain-exa/). You can install it with:


```python
%pip install -qU langchain-exa
```

In order to use the package, you will also need to set the `EXA_API_KEY` environment variable to your Exa API key.

## Retriever

You can use the [`ExaSearchRetriever`](/docs/integrations/tools/exa_search#using-exasearchretriever) in a standard retrieval pipeline. You can import it as follows


```python
from langchain_exa import ExaSearchRetriever
```

## Tools

You can use Exa as an agent tool as described in the [Exa tool calling docs](/docs/integrations/tools/exa_search#using-the-exa-sdk-as-langchain-agent-tools).

