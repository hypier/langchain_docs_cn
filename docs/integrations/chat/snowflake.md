---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/snowflake.ipynb
---
# Snowflake Cortex

[Snowflake Cortex](https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions) gives you instant access to industry-leading large language models (LLMs) trained by researchers at companies like Mistral, Reka, Meta, and Google, including [Snowflake Arctic](https://www.snowflake.com/en/data-cloud/arctic/), an open enterprise-grade model developed by Snowflake.

This example goes over how to use LangChain to interact with Snowflake Cortex.

### Installation and setup

We start by installing the `snowflake-snowpark-python` library, using the command below. Then we configure the credentials for connecting to Snowflake, as environment variables or pass them directly.


```python
%pip install --upgrade --quiet snowflake-snowpark-python
```
```output
Note: you may need to restart the kernel to use updated packages.
```

```python
import getpass
import os

# First step is to set up the environment variables, to connect to Snowflake,
# you can also pass these snowflake credentials while instantiating the model

if os.environ.get("SNOWFLAKE_ACCOUNT") is None:
    os.environ["SNOWFLAKE_ACCOUNT"] = getpass.getpass("Account: ")

if os.environ.get("SNOWFLAKE_USERNAME") is None:
    os.environ["SNOWFLAKE_USERNAME"] = getpass.getpass("Username: ")

if os.environ.get("SNOWFLAKE_PASSWORD") is None:
    os.environ["SNOWFLAKE_PASSWORD"] = getpass.getpass("Password: ")

if os.environ.get("SNOWFLAKE_DATABASE") is None:
    os.environ["SNOWFLAKE_DATABASE"] = getpass.getpass("Database: ")

if os.environ.get("SNOWFLAKE_SCHEMA") is None:
    os.environ["SNOWFLAKE_SCHEMA"] = getpass.getpass("Schema: ")

if os.environ.get("SNOWFLAKE_WAREHOUSE") is None:
    os.environ["SNOWFLAKE_WAREHOUSE"] = getpass.getpass("Warehouse: ")

if os.environ.get("SNOWFLAKE_ROLE") is None:
    os.environ["SNOWFLAKE_ROLE"] = getpass.getpass("Role: ")
```


```python
from langchain_community.chat_models import ChatSnowflakeCortex
from langchain_core.messages import HumanMessage, SystemMessage

# By default, we'll be using the cortex provided model: `snowflake-arctic`, with function: `complete`
chat = ChatSnowflakeCortex()
```

The above cell assumes that your Snowflake credentials are set in your environment variables. If you would rather manually specify them, use the following code:

```python
chat = ChatSnowflakeCortex(
    # change default cortex model and function
    model="snowflake-arctic",
    cortex_function="complete",

    # change default generation parameters
    temperature=0,
    max_tokens=10,
    top_p=0.95,

    # specify snowflake credentials
    account="YOUR_SNOWFLAKE_ACCOUNT",
    username="YOUR_SNOWFLAKE_USERNAME",
    password="YOUR_SNOWFLAKE_PASSWORD",
    database="YOUR_SNOWFLAKE_DATABASE",
    schema="YOUR_SNOWFLAKE_SCHEMA",
    role="YOUR_SNOWFLAKE_ROLE",
    warehouse="YOUR_SNOWFLAKE_WAREHOUSE"
)
```

### Calling the model
We can now call the model using the `invoke` or `generate` method.

#### Generation


```python
messages = [
    SystemMessage(content="You are a friendly assistant."),
    HumanMessage(content="What are large language models?"),
]
chat.invoke(messages)
```



```output
AIMessage(content=" Large language models are artificial intelligence systems designed to understand, generate, and manipulate human language. These models are typically based on deep learning techniques and are trained on vast amounts of text data to learn patterns and structures in language. They can perform a wide range of language-related tasks, such as language translation, text generation, sentiment analysis, and answering questions. Some well-known large language models include Google's BERT, OpenAI's GPT series, and Facebook's RoBERTa. These models have shown remarkable performance in various natural language processing tasks, and their applications continue to expand as research in AI progresses.", response_metadata={'completion_tokens': 131, 'prompt_tokens': 29, 'total_tokens': 160}, id='run-5435bd0a-83fd-4295-b237-66cbd1b5c0f3-0')
```


### Streaming
`ChatSnowflakeCortex` doesn't support streaming as of now. Support for streaming will be coming in the later versions!


## Related

- Chat model [conceptual guide](/docs/concepts/#chat-models)
- Chat model [how-to guides](/docs/how_to/#chat-models)
