---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/text_embeddings_inference.ipynb
---
# Text Embeddings Inference

>[Hugging Face Text Embeddings Inference (TEI)](https://huggingface.co/docs/text-embeddings-inference/index) is a toolkit for deploying and serving open-source
> text embeddings and sequence classification models. `TEI` enables high-performance extraction for the most popular models,
>including `FlagEmbedding`, `Ember`, `GTE` and `E5`.

To use it within langchain, first install `huggingface-hub`.


```python
%pip install --upgrade huggingface-hub
```

Then expose an embedding model using TEI. For instance, using Docker, you can serve `BAAI/bge-large-en-v1.5` as follows:

```bash
model=BAAI/bge-large-en-v1.5
revision=refs/pr/5
volume=$PWD/data # share a volume with the Docker container to avoid downloading weights every run

docker run --gpus all -p 8080:80 -v $volume:/data --pull always ghcr.io/huggingface/text-embeddings-inference:0.6 --model-id $model --revision $revision
```

Finally, instantiate the client and embed your texts.


```python
from langchain_huggingface.embeddings import HuggingFaceEndpointEmbeddings
```


```python
embeddings = HuggingFaceEndpointEmbeddings(model="http://localhost:8080")
```


```python
text = "What is deep learning?"
```


```python
query_result = embeddings.embed_query(text)
query_result[:3]
```



```output
[0.018113142, 0.00302585, -0.049911194]
```



```python
doc_result = embeddings.embed_documents([text])
```


```python
doc_result[0][:3]
```



```output
[0.018113142, 0.00302585, -0.049911194]
```



## Related

- Embedding model [conceptual guide](/docs/concepts/#embedding-models)
- Embedding model [how-to guides](/docs/how_to/#embedding-models)
