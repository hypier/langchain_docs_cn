---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/openvino.ipynb
---
# OpenVINO

[OpenVINO™](https://github.com/openvinotoolkit/openvino) is an open-source toolkit for optimizing and deploying AI inference. OpenVINO™ Runtime can enable running the same model optimized across various hardware [devices](https://github.com/openvinotoolkit/openvino?tab=readme-ov-file#supported-hardware-matrix). Accelerate your deep learning performance across use cases like: language + LLMs, computer vision, automatic speech recognition, and more.

OpenVINO models can be run locally through the `HuggingFacePipeline` [class](https://python.langchain.com/docs/integrations/llms/huggingface_pipeline). To deploy a model with OpenVINO, you can specify the `backend="openvino"` parameter to trigger OpenVINO as backend inference framework.

To use, you should have the ``optimum-intel`` with OpenVINO Accelerator python [package installed](https://github.com/huggingface/optimum-intel?tab=readme-ov-file#installation).


```python
%pip install --upgrade-strategy eager "optimum[openvino,nncf]" langchain-huggingface --quiet
```

### Model Loading

Models can be loaded by specifying the model parameters using the `from_model_id` method.

If you have an Intel GPU, you can specify `model_kwargs={"device": "GPU"}` to run inference on it.


```python
from langchain_huggingface import HuggingFacePipeline

ov_config = {"PERFORMANCE_HINT": "LATENCY", "NUM_STREAMS": "1", "CACHE_DIR": ""}

ov_llm = HuggingFacePipeline.from_model_id(
    model_id="gpt2",
    task="text-generation",
    backend="openvino",
    model_kwargs={"device": "CPU", "ov_config": ov_config},
    pipeline_kwargs={"max_new_tokens": 10},
)
```

They can also be loaded by passing in an existing [`optimum-intel`](https://huggingface.co/docs/optimum/main/en/intel/inference) pipeline directly


```python
from optimum.intel.openvino import OVModelForCausalLM
from transformers import AutoTokenizer, pipeline

model_id = "gpt2"
device = "CPU"
tokenizer = AutoTokenizer.from_pretrained(model_id)
ov_model = OVModelForCausalLM.from_pretrained(
    model_id, export=True, device=device, ov_config=ov_config
)
ov_pipe = pipeline(
    "text-generation", model=ov_model, tokenizer=tokenizer, max_new_tokens=10
)
ov_llm = HuggingFacePipeline(pipeline=ov_pipe)
```

### Create Chain

With the model loaded into memory, you can compose it with a prompt to
form a chain.


```python
from langchain_core.prompts import PromptTemplate

template = """Question: {question}

Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)

chain = prompt | ov_llm

question = "What is electroencephalography?"

print(chain.invoke({"question": question}))
```

To get response without prompt, you can bind `skip_prompt=True` with LLM.


```python
chain = prompt | ov_llm.bind(skip_prompt=True)

question = "What is electroencephalography?"

print(chain.invoke({"question": question}))
```

### Inference with local OpenVINO model

It is possible to [export your model](https://github.com/huggingface/optimum-intel?tab=readme-ov-file#export) to the OpenVINO IR format with the CLI, and load the model from local folder.



```python
!optimum-cli export openvino --model gpt2 ov_model_dir
```

It is recommended to apply 8 or 4-bit weight quantization to reduce inference latency and model footprint using `--weight-format`:


```python
!optimum-cli export openvino --model gpt2  --weight-format int8 ov_model_dir # for 8-bit quantization

!optimum-cli export openvino --model gpt2  --weight-format int4 ov_model_dir # for 4-bit quantization
```


```python
ov_llm = HuggingFacePipeline.from_model_id(
    model_id="ov_model_dir",
    task="text-generation",
    backend="openvino",
    model_kwargs={"device": "CPU", "ov_config": ov_config},
    pipeline_kwargs={"max_new_tokens": 10},
)

chain = prompt | ov_llm

question = "What is electroencephalography?"

print(chain.invoke({"question": question}))
```

You can get additional inference speed improvement with Dynamic Quantization of activations and KV-cache quantization. These options can be enabled with `ov_config` as follows:


```python
ov_config = {
    "KV_CACHE_PRECISION": "u8",
    "DYNAMIC_QUANTIZATION_GROUP_SIZE": "32",
    "PERFORMANCE_HINT": "LATENCY",
    "NUM_STREAMS": "1",
    "CACHE_DIR": "",
}
```

### Streaming

You can use `stream` method to get a streaming of LLM output, 


```python
generation_config = {"skip_prompt": True, "pipeline_kwargs": {"max_new_tokens": 100}}
chain = prompt | ov_llm.bind(**generation_config)

for chunk in chain.stream(question):
    print(chunk, end="", flush=True)
```

For more information refer to:

* [OpenVINO LLM guide](https://docs.openvino.ai/2024/learn-openvino/llm_inference_guide.html).

* [OpenVINO Documentation](https://docs.openvino.ai/2024/home.html).

* [OpenVINO Get Started Guide](https://www.intel.com/content/www/us/en/content-details/819067/openvino-get-started-guide.html).
  
* [RAG Notebook with LangChain](https://github.com/openvinotoolkit/openvino_notebooks/tree/latest/notebooks/llm-rag-langchain).


## Related

- LLM [conceptual guide](/docs/concepts/#llms)
- LLM [how-to guides](/docs/how_to/#llms)