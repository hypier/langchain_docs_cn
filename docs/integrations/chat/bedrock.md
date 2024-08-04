---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/bedrock.ipynb
sidebar_label: AWS Bedrock
---
# ChatBedrock

This doc will help you get started with AWS Bedrock [chat models](/docs/concepts/#chat-models). Amazon Bedrock is a fully managed service that offers a choice of high-performing foundation models (FMs) from leading AI companies like AI21 Labs, Anthropic, Cohere, Meta, Stability AI, and Amazon via a single API, along with a broad set of capabilities you need to build generative AI applications with security, privacy, and responsible AI. Using Amazon Bedrock, you can easily experiment with and evaluate top FMs for your use case, privately customize them with your data using techniques such as fine-tuning and Retrieval Augmented Generation (RAG), and build agents that execute tasks using your enterprise systems and data sources. Since Amazon Bedrock is serverless, you don't have to manage any infrastructure, and you can securely integrate and deploy generative AI capabilities into your applications using the AWS services you are already familiar with.

For more information on which models are accessible via Bedrock, head to the [AWS docs](https://docs.aws.amazon.com/bedrock/latest/userguide/models-features.html).

For detailed documentation of all ChatBedrock features and configurations head to the [API reference](https://api.python.langchain.com/en/latest/chat_models/langchain_aws.chat_models.bedrock.ChatBedrock.html).

## Overview
### Integration details

| Class | Package | Local | Serializable | [JS support](https://js.langchain.com/v0.2/docs/integrations/chat/bedrock) | Package downloads | Package latest |
| :--- | :--- | :---: | :---: |  :---: | :---: | :---: |
| [ChatBedrock](https://api.python.langchain.com/en/latest/chat_models/langchain_aws.chat_models.bedrock.ChatBedrock.html) | [langchain-aws](https://api.python.langchain.com/en/latest/aws_api_reference.html) | ❌ | beta | ✅ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain-aws?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain-aws?style=flat-square&label=%20) |

### Model features
| [Tool calling](/docs/how_to/tool_calling) | [Structured output](/docs/how_to/structured_output/) | JSON mode | [Image input](/docs/how_to/multimodal_inputs/) | Audio input | Video input | [Token-level streaming](/docs/how_to/chat_streaming/) | Native async | [Token usage](/docs/how_to/chat_token_usage_tracking/) | [Logprobs](/docs/how_to/logprobs/) |
| :---: | :---: | :---: | :---: |  :---: | :---: | :---: | :---: | :---: | :---: |
| ✅ | ✅ | ❌ | ✅ | ❌ | ❌ | ✅ | ❌ | ✅ | ❌ | 

## Setup

To access Bedrock models you'll need to create an AWS account, set up the Bedrock API service, get an access key ID and secret key, and install the `langchain-aws` integration package.

### Credentials

Head to the [AWS docs](https://docs.aws.amazon.com/bedrock/latest/userguide/setting-up.html) to sign up to AWS and setup your credentials. You'll also need to turn on model access for your account, which you can do by following [these instructions](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html).

If you want to get automated tracing of your model calls you can also set your [LangSmith](https://docs.smith.langchain.com/) API key by uncommenting below:


```python
# os.environ["LANGSMITH_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
# os.environ["LANGSMITH_TRACING"] = "true"
```

### Installation

The LangChain Bedrock integration lives in the `langchain-aws` package:


```python
%pip install -qU langchain-aws
```

## Instantiation

Now we can instantiate our model object and generate chat completions:


```python
from langchain_aws import ChatBedrock

llm = ChatBedrock(
    model_id="anthropic.claude-3-sonnet-20240229-v1:0",
    model_kwargs=dict(temperature=0),
    # other params...
)
```

## Invocation


```python
messages = [
    (
        "system",
        "You are a helpful assistant that translates English to French. Translate the user sentence.",
    ),
    ("human", "I love programming."),
]
ai_msg = llm.invoke(messages)
ai_msg
```



```output
AIMessage(content="Voici la traduction en français :\n\nJ'aime la programmation.", additional_kwargs={'usage': {'prompt_tokens': 29, 'completion_tokens': 21, 'total_tokens': 50}, 'stop_reason': 'end_turn', 'model_id': 'anthropic.claude-3-sonnet-20240229-v1:0'}, response_metadata={'usage': {'prompt_tokens': 29, 'completion_tokens': 21, 'total_tokens': 50}, 'stop_reason': 'end_turn', 'model_id': 'anthropic.claude-3-sonnet-20240229-v1:0'}, id='run-fdb07dc3-ff72-430d-b22b-e7824b15c766-0', usage_metadata={'input_tokens': 29, 'output_tokens': 21, 'total_tokens': 50})
```



```python
print(ai_msg.content)
```
```output
Voici la traduction en français :

J'aime la programmation.
```
## Chaining

We can [chain](/docs/how_to/sequence/) our model with a prompt template like so:


```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant that translates {input_language} to {output_language}.",
        ),
        ("human", "{input}"),
    ]
)

chain = prompt | llm
chain.invoke(
    {
        "input_language": "English",
        "output_language": "German",
        "input": "I love programming.",
    }
)
```



```output
AIMessage(content='Ich liebe Programmieren.', additional_kwargs={'usage': {'prompt_tokens': 23, 'completion_tokens': 11, 'total_tokens': 34}, 'stop_reason': 'end_turn', 'model_id': 'anthropic.claude-3-sonnet-20240229-v1:0'}, response_metadata={'usage': {'prompt_tokens': 23, 'completion_tokens': 11, 'total_tokens': 34}, 'stop_reason': 'end_turn', 'model_id': 'anthropic.claude-3-sonnet-20240229-v1:0'}, id='run-5ad005ce-9f31-4670-baa0-9373d418698a-0', usage_metadata={'input_tokens': 23, 'output_tokens': 11, 'total_tokens': 34})
```


## ***Beta***: Bedrock Converse API

AWS has recently recently the Bedrock Converse API which provides a unified conversational interface for Bedrock models. This API does not yet support custom models. You can see a list of all [models that are supported here](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html). To improve reliability the ChatBedrock integration will switch to using the Bedrock Converse API as soon as it has feature parity with the existing Bedrock API. Until then a separate [ChatBedrockConverse](https://api.python.langchain.com/en/latest/chat_models/langchain_aws.chat_models.bedrock_converse.ChatBedrockConverse.html#langchain_aws.chat_models.bedrock_converse.ChatBedrockConverse) integration has been released in beta for users who do not need to use custom models.

You can use it like so:


```python
from langchain_aws import ChatBedrockConverse

llm = ChatBedrockConverse(
    model="anthropic.claude-3-sonnet-20240229-v1:0",
    temperature=0,
    max_tokens=None,
    # other params...
)

llm.invoke(messages)
```
```output
/Users/bagatur/langchain/libs/core/langchain_core/_api/beta_decorator.py:87: LangChainBetaWarning: The class `ChatBedrockConverse` is in beta. It is actively being worked on, so the API may change.
  warn_beta(
```


```output
AIMessage(content="Voici la traduction en français :\n\nJ'aime la programmation.", response_metadata={'ResponseMetadata': {'RequestId': '122fb1c8-c3c5-4b06-941e-c95d210bfbc7', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Mon, 01 Jul 2024 21:48:25 GMT', 'content-type': 'application/json', 'content-length': '243', 'connection': 'keep-alive', 'x-amzn-requestid': '122fb1c8-c3c5-4b06-941e-c95d210bfbc7'}, 'RetryAttempts': 0}, 'stopReason': 'end_turn', 'metrics': {'latencyMs': 830}}, id='run-0e3df22f-fcd8-4fbb-a4fb-565227e7e430-0', usage_metadata={'input_tokens': 29, 'output_tokens': 21, 'total_tokens': 50})
```


## API reference

For detailed documentation of all ChatBedrock features and configurations head to the API reference: https://api.python.langchain.com/en/latest/chat_models/langchain_aws.chat_models.bedrock.ChatBedrock.html

For detailed documentation of all ChatBedrockConverse features and configurations head to the API reference: https://api.python.langchain.com/en/latest/chat_models/langchain_aws.chat_models.bedrock_converse.ChatBedrockConverse.html


## Related

- Chat model [conceptual guide](/docs/concepts/#chat-models)
- Chat model [how-to guides](/docs/how_to/#chat-models)
