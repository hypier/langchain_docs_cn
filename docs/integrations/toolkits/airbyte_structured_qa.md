---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/airbyte_structured_qa.ipynb
---
# Airbyte Question Answering
This notebook shows how to do question answering over structured data, in this case using the `AirbyteStripeLoader`.

Vectorstores often have a hard time answering questions that requires computing, grouping and filtering structured data so the high level idea is to use a `pandas` dataframe to help with these types of questions. 


```python
%pip install -qU langchain-community
```

1. Load data from Stripe using Airbyte. user the `record_handler` paramater to return a JSON from the data loader.


```python
import os

import pandas as pd
from langchain.agents import AgentType
from langchain_community.document_loaders.airbyte import AirbyteStripeLoader
from langchain_experimental.agents import create_pandas_dataframe_agent
from langchain_openai import ChatOpenAI

stream_name = "customers"
config = {
    "client_secret": os.getenv("STRIPE_CLIENT_SECRET"),
    "account_id": os.getenv("STRIPE_ACCOUNT_D"),
    "start_date": "2023-01-20T00:00:00Z",
}


def handle_record(record: dict, _id: str):
    return record.data


loader = AirbyteStripeLoader(
    config=config,
    record_handler=handle_record,
    stream_name=stream_name,
)
data = loader.load()
```

2. Pass the data to `pandas` dataframe.


```python
df = pd.DataFrame(data)
```

3. Pass the dataframe `df` to the `create_pandas_dataframe_agent` and invoke



```python
agent = create_pandas_dataframe_agent(
    ChatOpenAI(temperature=0, model="gpt-4"),
    df,
    verbose=True,
    agent_type=AgentType.OPENAI_FUNCTIONS,
)
```

4. Run the agent


```python
output = agent.run("How many rows are there?")
```
