---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/ibm_watsonx.ipynb
---
# IBM watsonx.ai

>[WatsonxLLM](https://ibm.github.io/watsonx-ai-python-sdk/fm_extensions.html#langchain) is a wrapper for IBM [watsonx.ai](https://www.ibm.com/products/watsonx-ai) foundation models.

This example shows how to communicate with `watsonx.ai` models using `LangChain`.

## Setting up

Install the package `langchain-ibm`.


```python
!pip install -qU langchain-ibm
```

This cell defines the WML credentials required to work with watsonx Foundation Model inferencing.

**Action:** Provide the IBM Cloud user API key. For details, see
[documentation](https://cloud.ibm.com/docs/account?topic=account-userapikey&interface=ui).


```python
import os
from getpass import getpass

watsonx_api_key = getpass()
os.environ["WATSONX_APIKEY"] = watsonx_api_key
```

Additionaly you are able to pass additional secrets as an environment variable. 


```python
import os

os.environ["WATSONX_URL"] = "your service instance url"
os.environ["WATSONX_TOKEN"] = "your token for accessing the CPD cluster"
os.environ["WATSONX_PASSWORD"] = "your password for accessing the CPD cluster"
os.environ["WATSONX_USERNAME"] = "your username for accessing the CPD cluster"
os.environ["WATSONX_INSTANCE_ID"] = "your instance_id for accessing the CPD cluster"
```

## Load the model

You might need to adjust model `parameters` for different models or tasks. For details, refer to [documentation](https://ibm.github.io/watsonx-ai-python-sdk/fm_model.html#metanames.GenTextParamsMetaNames).


```python
parameters = {
    "decoding_method": "sample",
    "max_new_tokens": 100,
    "min_new_tokens": 1,
    "temperature": 0.5,
    "top_k": 50,
    "top_p": 1,
}
```

Initialize the `WatsonxLLM` class with previously set parameters.


**Note**: 

- To provide context for the API call, you must add `project_id` or `space_id`. For more information see [documentation](https://www.ibm.com/docs/en/watsonx-as-a-service?topic=projects).
- Depending on the region of your provisioned service instance, use one of the urls described [here](https://ibm.github.io/watsonx-ai-python-sdk/setup_cloud.html#authentication).

In this example, we’ll use the `project_id` and Dallas url.


You need to specify `model_id` that will be used for inferencing. All available models you can find in [documentation](https://ibm.github.io/watsonx-ai-python-sdk/fm_model.html#ibm_watsonx_ai.foundation_models.utils.enums.ModelTypes).


```python
from langchain_ibm import WatsonxLLM

watsonx_llm = WatsonxLLM(
    model_id="ibm/granite-13b-instruct-v2",
    url="https://us-south.ml.cloud.ibm.com",
    project_id="PASTE YOUR PROJECT_ID HERE",
    params=parameters,
)
```

Alternatively you can use Cloud Pak for Data credentials. For details, see [documentation](https://ibm.github.io/watsonx-ai-python-sdk/setup_cpd.html).    


```python
watsonx_llm = WatsonxLLM(
    model_id="ibm/granite-13b-instruct-v2",
    url="PASTE YOUR URL HERE",
    username="PASTE YOUR USERNAME HERE",
    password="PASTE YOUR PASSWORD HERE",
    instance_id="openshift",
    version="4.8",
    project_id="PASTE YOUR PROJECT_ID HERE",
    params=parameters,
)
```

Instead of `model_id`, you can also pass the `deployment_id` of the previously tuned model. The entire model tuning workflow is described [here](https://ibm.github.io/watsonx-ai-python-sdk/pt_working_with_class_and_prompt_tuner.html).


```python
watsonx_llm = WatsonxLLM(
    deployment_id="PASTE YOUR DEPLOYMENT_ID HERE",
    url="https://us-south.ml.cloud.ibm.com",
    project_id="PASTE YOUR PROJECT_ID HERE",
    params=parameters,
)
```

For certain requirements, there is an option to pass the IBM's [`APIClient`](https://ibm.github.io/watsonx-ai-python-sdk/base.html#apiclient) object into the `WatsonxLLM` class.


```python
from ibm_watsonx_ai import APIClient

api_client = APIClient(...)

watsonx_llm = WatsonxLLM(
    model_id="ibm/granite-13b-instruct-v2",
    watsonx_client=api_client,
)
```

You can also pass the IBM's [`ModelInference`](https://ibm.github.io/watsonx-ai-python-sdk/fm_model_inference.html) object into the `WatsonxLLM` class.


```python
from ibm_watsonx_ai.foundation_models import ModelInference

model = ModelInference(...)

watsonx_llm = WatsonxLLM(watsonx_model=model)
```

## Create Chain
Create `PromptTemplate` objects which will be responsible for creating a random question.


```python
from langchain_core.prompts import PromptTemplate

template = "Generate a random question about {topic}: Question: "

prompt = PromptTemplate.from_template(template)
```

Provide a topic and run the chain.


```python
llm_chain = prompt | watsonx_llm

topic = "dog"

llm_chain.invoke(topic)
```



```output
'What is the difference between a dog and a wolf?'
```


## Calling the Model Directly
To obtain completions, you can call the model directly using a string prompt.


```python
# Calling a single prompt

watsonx_llm.invoke("Who is man's best friend?")
```



```output
"Man's best friend is his dog. "
```



```python
# Calling multiple prompts

watsonx_llm.generate(
    [
        "The fastest dog in the world?",
        "Describe your chosen dog breed",
    ]
)
```



```output
LLMResult(generations=[[Generation(text='The fastest dog in the world is the greyhound, which can run up to 45 miles per hour. This is about the same speed as a human running down a track. Greyhounds are very fast because they have long legs, a streamlined body, and a strong tail. They can run this fast for short distances, but they can also run for long distances, like a marathon. ', generation_info={'finish_reason': 'eos_token'})], [Generation(text='The Beagle is a scent hound, meaning it is bred to hunt by following a trail of scents.', generation_info={'finish_reason': 'eos_token'})]], llm_output={'token_usage': {'generated_token_count': 106, 'input_token_count': 13}, 'model_id': 'ibm/granite-13b-instruct-v2', 'deployment_id': ''}, run=[RunInfo(run_id=UUID('52cb421d-b63f-4c5f-9b04-d4770c664725')), RunInfo(run_id=UUID('df2ea606-1622-4ed7-8d5d-8f6e068b71c4'))])
```


## Streaming the Model output 

You can stream the model output.


```python
for chunk in watsonx_llm.stream(
    "Describe your favorite breed of dog and why it is your favorite."
):
    print(chunk, end="")
```
```output
My favorite breed of dog is a Labrador Retriever. Labradors are my favorite because they are extremely smart, very friendly, and love to be with people. They are also very playful and love to run around and have a lot of energy.
```

## Related

- LLM [conceptual guide](/docs/concepts/#llms)
- LLM [how-to guides](/docs/how_to/#llms)
