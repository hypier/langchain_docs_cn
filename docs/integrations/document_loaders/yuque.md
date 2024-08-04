---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/yuque.ipynb
---
# Yuque

>[Yuque](https://www.yuque.com/) is a professional cloud-based knowledge base for team collaboration in documentation.

This notebook covers how to load documents from `Yuque`.

You can obtain the personal access token by clicking on your personal avatar in the [Personal Settings](https://www.yuque.com/settings/tokens) page.


```python
from langchain_community.document_loaders import YuqueLoader
```


```python
loader = YuqueLoader(access_token="<your_personal_access_token>")
```


```python
docs = loader.load()
```


## Related

- Document loader [conceptual guide](/docs/concepts/#document-loaders)
- Document loader [how-to guides](/docs/how_to/#document-loaders)
