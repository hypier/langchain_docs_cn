---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/whatsapp_chat.ipynb
---
# WhatsApp Chat

>[WhatsApp](https://www.whatsapp.com/) (also called `WhatsApp Messenger`) is a freeware, cross-platform, centralized instant messaging (IM) and voice-over-IP (VoIP) service. It allows users to send text and voice messages, make voice and video calls, and share images, documents, user locations, and other content.

This notebook covers how to load data from the `WhatsApp Chats` into a format that can be ingested into LangChain.


```python
from langchain_community.document_loaders import WhatsAppChatLoader
```


```python
loader = WhatsAppChatLoader("example_data/whatsapp_chat.txt")
```


```python
loader.load()
```


## Related

- Document loader [conceptual guide](/docs/concepts/#document-loaders)
- Document loader [how-to guides](/docs/how_to/#document-loaders)
