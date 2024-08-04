---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/xml.ipynb
---
# XML

The `UnstructuredXMLLoader` is used to load `XML` files. The loader works with `.xml` files. The page content will be the text extracted from the XML tags.


```python
from langchain_community.document_loaders import UnstructuredXMLLoader

loader = UnstructuredXMLLoader(
    "./example_data/factbook.xml",
)
docs = loader.load()
docs[0]
```


## Related

- Document loader [conceptual guide](/docs/concepts/#document-loaders)
- Document loader [how-to guides](/docs/how_to/#document-loaders)
