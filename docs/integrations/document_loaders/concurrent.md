---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/concurrent.ipynb
---
# Concurrent Loader

Works just like the GenericLoader but concurrently for those who choose to optimize their workflow.



```python
from langchain_community.document_loaders import ConcurrentLoader
```


```python
loader = ConcurrentLoader.from_filesystem("example_data/", glob="**/*.txt")
```


```python
files = loader.load()
```


```python
len(files)
```



```output
2
```



## Related

- Document loader [conceptual guide](/docs/concepts/#document-loaders)
- Document loader [how-to guides](/docs/how_to/#document-loaders)
