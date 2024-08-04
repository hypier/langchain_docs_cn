---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/acreom.ipynb
---
# acreom

[acreom](https://acreom.com) is a dev-first knowledge base with tasks running on local markdown files.

Below is an example on how to load a local acreom vault into Langchain. As the local vault in acreom is a folder of plain text .md files, the loader requires the path to the directory. 

Vault files may contain some metadata which is stored as a YAML header. These values will be added to the document’s metadata if `collect_metadata` is set to true. 


```python
from langchain_community.document_loaders import AcreomLoader
```


```python
loader = AcreomLoader("<path-to-acreom-vault>", collect_metadata=False)
```


```python
docs = loader.load()
```


## Related

- Document loader [conceptual guide](/docs/concepts/#document-loaders)
- Document loader [how-to guides](/docs/how_to/#document-loaders)
