---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/solar.ipynb
---
```python
import os

os.environ["SOLAR_API_KEY"] = "SOLAR_API_KEY"

from langchain_community.chat_models.solar import SolarChat
from langchain_core.messages import HumanMessage, SystemMessage

chat = SolarChat(max_tokens=1024)

messages = [
    SystemMessage(
        content="你是一个有帮助的助手，负责将英语翻译成韩语。"
    ),
    HumanMessage(
        content="将这句话从英语翻译成韩语。我想建立一个大型语言模型的项目。"
    ),
]

chat.invoke(messages)
```



```output
AIMessage(content='저는 대형 언어 모델 프로젝트를 구축하고 싶습니다.')
```

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)