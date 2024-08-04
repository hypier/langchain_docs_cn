---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/callbacks/confident.ipynb
---
# Confident

>[DeepEval](https://confident-ai.com) package for unit testing LLMs.
> Using Confident, everyone can build robust language models through faster iterations
> using both unit testing and integration testing. We provide support for each step in the iteration
> from synthetic data creation to testing.


In this guide we will demonstrate how to test and measure LLMs in performance. We show how you can use our callback to measure performance and how you can define your own metric and log them into our dashboard.

DeepEval also offers:
- How to generate synthetic data
- How to measure performance
- A dashboard to monitor and review results over time

## Installation and Setup


```python
%pip install --upgrade --quiet  langchain langchain-openai langchain-community deepeval langchain-chroma
```

### Getting API Credentials

To get the DeepEval API credentials, follow the next steps:

1. Go to https://app.confident-ai.com
2. Click on "Organization"
3. Copy the API Key.


When you log in, you will also be asked to set the `implementation` name. The implementation name is required to describe the type of implementation. (Think of what you want to call your project. We recommend making it descriptive.)


```python
!deepeval login
```

### Setup DeepEval

You can, by default, use the `DeepEvalCallbackHandler` to set up the metrics you want to track. However, this has limited support for metrics at the moment (more to be added soon). It currently supports:
- [Answer Relevancy](https://docs.confident-ai.com/docs/measuring_llm_performance/answer_relevancy)
- [Bias](https://docs.confident-ai.com/docs/measuring_llm_performance/debias)
- [Toxicness](https://docs.confident-ai.com/docs/measuring_llm_performance/non_toxic)


```python
from deepeval.metrics.answer_relevancy import AnswerRelevancy

# Here we want to make sure the answer is minimally relevant
answer_relevancy_metric = AnswerRelevancy(minimum_score=0.5)
```

## Get Started

To use the `DeepEvalCallbackHandler`, we need the `implementation_name`. 


```python
from langchain_community.callbacks.confident_callback import DeepEvalCallbackHandler

deepeval_callback = DeepEvalCallbackHandler(
    implementation_name="langchainQuickstart", metrics=[answer_relevancy_metric]
)
```

### Scenario 1: Feeding into LLM

You can then feed it into your LLM with OpenAI.


```python
from langchain_openai import OpenAI

llm = OpenAI(
    temperature=0,
    callbacks=[deepeval_callback],
    verbose=True,
    openai_api_key="<YOUR_API_KEY>",
)
output = llm.generate(
    [
        "What is the best evaluation tool out there? (no bias at all)",
    ]
)
```



```output
LLMResult(generations=[[Generation(text='\n\nQ: What did the fish say when he hit the wall? \nA: Dam.', generation_info={'finish_reason': 'stop', 'logprobs': None})], [Generation(text='\n\nThe Moon \n\nThe moon is high in the midnight sky,\nSparkling like a star above.\nThe night so peaceful, so serene,\nFilling up the air with love.\n\nEver changing and renewing,\nA never-ending light of grace.\nThe moon remains a constant view,\nA reminder of life’s gentle pace.\n\nThrough time and space it guides us on,\nA never-fading beacon of hope.\nThe moon shines down on us all,\nAs it continues to rise and elope.', generation_info={'finish_reason': 'stop', 'logprobs': None})], [Generation(text='\n\nQ. What did one magnet say to the other magnet?\nA. "I find you very attractive!"', generation_info={'finish_reason': 'stop', 'logprobs': None})], [Generation(text="\n\nThe world is charged with the grandeur of God.\nIt will flame out, like shining from shook foil;\nIt gathers to a greatness, like the ooze of oil\nCrushed. Why do men then now not reck his rod?\n\nGenerations have trod, have trod, have trod;\nAnd all is seared with trade; bleared, smeared with toil;\nAnd wears man's smudge and shares man's smell: the soil\nIs bare now, nor can foot feel, being shod.\n\nAnd for all this, nature is never spent;\nThere lives the dearest freshness deep down things;\nAnd though the last lights off the black West went\nOh, morning, at the brown brink eastward, springs —\n\nBecause the Holy Ghost over the bent\nWorld broods with warm breast and with ah! bright wings.\n\n~Gerard Manley Hopkins", generation_info={'finish_reason': 'stop', 'logprobs': None})], [Generation(text='\n\nQ: What did one ocean say to the other ocean?\nA: Nothing, they just waved.', generation_info={'finish_reason': 'stop', 'logprobs': None})], [Generation(text="\n\nA poem for you\n\nOn a field of green\n\nThe sky so blue\n\nA gentle breeze, the sun above\n\nA beautiful world, for us to love\n\nLife is a journey, full of surprise\n\nFull of joy and full of surprise\n\nBe brave and take small steps\n\nThe future will be revealed with depth\n\nIn the morning, when dawn arrives\n\nA fresh start, no reason to hide\n\nSomewhere down the road, there's a heart that beats\n\nBelieve in yourself, you'll always succeed.", generation_info={'finish_reason': 'stop', 'logprobs': None})]], llm_output={'token_usage': {'completion_tokens': 504, 'total_tokens': 528, 'prompt_tokens': 24}, 'model_name': 'text-davinci-003'})
```


You can then check the metric if it was successful by calling the `is_successful()` method.


```python
answer_relevancy_metric.is_successful()
# returns True/False
```

Once you have ran that, you should be able to see our dashboard below. 

![Dashboard](https://docs.confident-ai.com/assets/images/dashboard-screenshot-b02db73008213a211b1158ff052d969e.png)

### Scenario 2: Tracking an LLM in a chain without callbacks

To track an LLM in a chain without callbacks, you can plug into it at the end.

We can start by defining a simple chain as shown below.


```python
import requests
from langchain.chains import RetrievalQA
from langchain_chroma import Chroma
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAI, OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter

text_file_url = "https://raw.githubusercontent.com/hwchase17/chat-your-data/master/state_of_the_union.txt"

openai_api_key = "sk-XXX"

with open("state_of_the_union.txt", "w") as f:
    response = requests.get(text_file_url)
    f.write(response.text)

loader = TextLoader("state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
texts = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings(openai_api_key=openai_api_key)
docsearch = Chroma.from_documents(texts, embeddings)

qa = RetrievalQA.from_chain_type(
    llm=OpenAI(openai_api_key=openai_api_key),
    chain_type="stuff",
    retriever=docsearch.as_retriever(),
)

# Providing a new question-answering pipeline
query = "Who is the president?"
result = qa.run(query)
```

After defining a chain, you can then manually check for answer similarity.


```python
answer_relevancy_metric.measure(result, query)
answer_relevancy_metric.is_successful()
```

### What's next?

You can create your own custom metrics [here](https://docs.confident-ai.com/docs/quickstart/custom-metrics). 

DeepEval also offers other features such as being able to [automatically create unit tests](https://docs.confident-ai.com/docs/quickstart/synthetic-data-creation), [tests for hallucination](https://docs.confident-ai.com/docs/measuring_llm_performance/factual_consistency).

If you are interested, check out our Github repository here [https://github.com/confident-ai/deepeval](https://github.com/confident-ai/deepeval). We welcome any PRs and discussions on how to improve LLM performance.