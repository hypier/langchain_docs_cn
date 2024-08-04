---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/databricks.ipynb
---
# Databricks

> [Databricks](https://www.databricks.com/) Lakehouse Platform unifies data, analytics, and AI on one platform.

This notebook provides a quick overview for getting started with Databricks [embedding models](/docs/concepts/#embedding-models). For detailed documentation of all DatabricksEmbeddings features and configurations head to the [API reference](https://api.python.langchain.com/en/latest/embeddings/langchain_community.embeddings.databricks.DatabricksEmbeddings.html).



## Overview

`DatabricksEmbeddings` class wraps an embedding model endpoint hosted on [Databricks Model Serving](https://docs.databricks.com/en/machine-learning/model-serving/index.html). This example notebook shows how to wrap your serving endpoint and use it as a embedding model in your LangChain application.


### Supported Methods

`DatabricksEmbeddings` supports all methods of `Embeddings` class including async APIs.


### Endpoint Requirement

The serving endpoint `DatabricksEmbeddings` wraps must have OpenAI-compatible embedding input/output format ([reference](https://mlflow.org/docs/latest/llms/deployments/index.html#embeddings)). As long as the input format is compatible, `DatabricksEmbeddings` can be used for any endpoint type hosted on [Databricks Model Serving](https://docs.databricks.com/en/machine-learning/model-serving/index.html):

1. Foundation Models - Curated list of state-of-the-art foundation models such as BAAI General Embedding (BGE). These endpoint are ready to use in your Databricks workspace without any set up.
2. Custom Models - You can also deploy custom embedding models to a serving endpoint via MLflow with
your choice of framework such as LangChain, Pytorch, Transformers, etc.
3. External Models - Databricks endpoints can serve models that are hosted outside Databricks as a proxy, such as proprietary model service like OpenAI text-embedding-3.


## Setup

To access Databricks models you'll need to create a Databricks account, set up credentials (only if you are outside Databricks workspace), and install required packages.

### Credentials (only if you are outside Databricks)

If you are running LangChain app inside Databricks, you can skip this step.

Otherwise, you need manually set the Databricks workspace hostname and personal access token to `DATABRICKS_HOST` and `DATABRICKS_TOKEN` environment variables, respectively. See [Authentication Documentation](https://docs.databricks.com/en/dev-tools/auth/index.html#databricks-personal-access-tokens) for how to get an access token.


```python
import getpass
import os

os.environ["DATABRICKS_HOST"] = "https://your-workspace.cloud.databricks.com"
os.environ["DATABRICKS_TOKEN"] = getpass.getpass("Enter your Databricks access token: ")
```

### Installation

The LangChain Databricks integration lives in the `langchain-community` package. Also, `mlflow >= 2.9 ` is required to run the code in this notebook.


```python
%pip install -qU langchain-community mlflow>=2.9.0
```

We first demonstrates how to query BGE model hosted as Foundation Models endpoint with `DatabricksEmbeddings`.

For other type of endpoints, there are some difference in how to set up the endpoint itself, however, once the endpoint is ready, there is no difference in how to query it.

## Instantiation


```python
from langchain_community.embeddings import DatabricksEmbeddings

embeddings = DatabricksEmbeddings(
    endpoint="databricks-bge-large-en",
    # Specify parameters for embedding queries and documents if needed
    # query_params={...},
    # document_params={...},
)
```

## Embed single text


```python
embeddings.embed_query("hello")[:3]
```
```output
[0.051055908203125, 0.007221221923828125, 0.003879547119140625]
```
## Embed documents


```python
documents = ["This is a dummy document.", "This is another dummy document."]
response = embeddings.embed_documents(documents)
print([e[:3] for e in response])  # Show first 3 elements of each embedding
```

## Wrapping Other Types of Endpoints

The example above uses an embedding model hosted as a Foundation Models API. To learn about how to use the other endpoint types, please refer to the documentation for `ChatDatabricks`. While the model type is different, required steps are the same.

* [Custom Model Endpoint](https://python.langchain.com/v0.2/docs/integrations/chat/databricks/#wrapping-custom-model-endpoint)
* [External Models](https://python.langchain.com/v0.2/docs/integrations/chat/databricks/#wrapping-external-models)

## API reference

For detailed documentation of all ChatDatabricks features and configurations head to the API reference: https://api.python.langchain.com/en/latest/embeddings/langchain_community.embeddings.databricks.DatabricksEmbeddings.html


## Related

- Embedding model [conceptual guide](/docs/concepts/#embedding-models)
- Embedding model [how-to guides](/docs/how_to/#embedding-models)
