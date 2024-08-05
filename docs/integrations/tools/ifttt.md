---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/ifttt.ipynb
---

# IFTTT WebHooks

本笔记本展示了如何使用 IFTTT Webhooks。

来自 https://github.com/SidU/teams-langchain-js/wiki/Connecting-IFTTT-Services.

## 创建一个 webhook
- 前往 https://ifttt.com/create

## 配置 "If This"
- 在 IFTTT 界面中点击 "If This" 按钮。
- 在搜索栏中搜索 "Webhooks"。
- 选择第一个选项 "接收带有 JSON 负载的网络请求"。
- 选择一个特定于您计划连接的服务的事件名称。这将使您更容易管理 webhook URL。例如，如果您连接的是 Spotify，可以使用 "Spotify" 作为您的事件名称。
- 点击 "创建触发器" 按钮以保存您的设置并创建您的 webhook。

## 配置 "Then That"
- 点击 IFTTT 界面中的 "Then That" 按钮。
- 搜索您想要连接的服务，例如 Spotify。
- 从服务中选择一个操作，例如 "将曲目添加到播放列表"。
- 通过指定必要的细节来配置操作，例如播放列表名称，例如 "来自 AI 的歌曲"。
- 在您的操作中引用通过 Webhook 接收到的 JSON Payload。对于 Spotify 场景，选择 "{{JsonPayload}}" 作为您的搜索查询。
- 点击 "创建操作" 按钮以保存您的操作设置。
- 配置操作完成后，点击 "完成" 按钮以完成设置。
- 恭喜！您已成功将 Webhook 连接到所需服务，准备开始接收数据和触发操作 🎉

## 完成
- 要获取您的 webhook URL，请访问 https://ifttt.com/maker_webhooks/settings
- 从那里复制 IFTTT 密钥值。URL 的格式为 https://maker.ifttt.com/use/YOUR_IFTTT_KEY。获取 YOUR_IFTTT_KEY 值。



```python
%pip install --upgrade --quiet  langchain-community
```


```python
from langchain_community.tools.ifttt import IFTTTWebhook
```


```python
import os

key = os.environ["IFTTTKey"]
url = f"https://maker.ifttt.com/trigger/spotify/json/with/key/{key}"
tool = IFTTTWebhook(
    name="Spotify", description="Add a song to spotify playlist", url=url
)
```


```python
tool.run("taylor swift")
```



```output
"Congratulations! You've fired the spotify JSON event"
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [使用指南](/docs/how_to/#tools)