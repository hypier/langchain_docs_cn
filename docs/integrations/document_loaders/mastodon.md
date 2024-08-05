---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/mastodon.ipynb
---

# Mastodon

>[Mastodon](https://joinmastodon.org/) 是一个联邦社交媒体和社交网络服务。

这个加载器使用 `Mastodon.py` Python 包从一系列 `Mastodon` 账户的 "toots" 中获取文本。

公共账户可以在没有任何身份验证的情况下进行查询。若要查询非公共账户或实例，您必须为您的账户注册一个应用程序，这样可以获得访问令牌，并设置该令牌和您账户的 API 基础 URL。

然后，您需要以 `@account@instance` 格式传入要提取的 Mastodon 账户名称。


```python
from langchain_community.document_loaders import MastodonTootsLoader
```


```python
%pip install --upgrade --quiet  Mastodon.py
```


```python
loader = MastodonTootsLoader(
    mastodon_accounts=["@Gargron@mastodon.social"],
    number_toots=50,  # 默认值为 100
)

# 或设置访问信息以使用 Mastodon 应用程序。
# 请注意，访问令牌可以传递给构造函数，
# 或者您可以设置环境变量 "MASTODON_ACCESS_TOKEN"。
# loader = MastodonTootsLoader(
#     access_token="<Mastodon 应用的访问令牌>",
#     api_base_url="<Mastodon 应用实例的 API 基础 URL>",
#     mastodon_accounts=["@Gargron@mastodon.social"],
#     number_toots=50,  # 默认值为 100
# )
```


```python
documents = loader.load()
for doc in documents[:3]:
    print(doc.page_content)
    print("=" * 80)
```
```output
<p>It is tough to leave this behind and go back to reality. And some people live here! I’m sure there are downsides but it sounds pretty good to me right now.</p>
================================================================================
<p>I wish we could stay here a little longer, but it is time to go home 🥲</p>
================================================================================
<p>Last day of the honeymoon. And it’s <a href="https://mastodon.social/tags/caturday" class="mention hashtag" rel="tag">#<span>caturday</span></a>! This cute tabby came to the restaurant to beg for food and got some chicken.</p>
================================================================================
```
toot 文本（文档的 `page_content`）默认是以 Mastodon API 返回的 HTML 格式。

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)