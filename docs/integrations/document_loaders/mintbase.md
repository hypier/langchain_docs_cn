---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/mintbase.ipynb
---

# Near 区块链

## 概述

本笔记本的目的是提供一种测试Langchain Document Loader在Near区块链上功能的方法。

最初，该加载器支持：

*   从NFT智能合约加载NFT作为文档（NEP-171和NEP-177）
*   Near主网，Near测试网（默认是主网）
*   Mintbase的Graph API

如果社区认为该加载器有价值，可以进行扩展。具体来说：

*   可以添加额外的API（例如，与交易相关的API）

该文档加载器需要：

*   一个免费的 [Mintbase API密钥](https://docs.mintbase.xyz/dev/mintbase-graph/)

输出格式如下：

- pageContent= 单个NFT
- metadata={'source': 'nft.yearofchef.near', 'blockchain': 'mainnet', 'tokenId': '1846'}

## 将NFT加载到文档加载器


```python
# get MINTBASE_API_KEY from https://docs.mintbase.xyz/dev/mintbase-graph/

mintbaseApiKey = "..."
```

### 选项 1：以太坊主网（默认 BlockchainType）


```python
from MintbaseLoader import MintbaseDocumentLoader

contractAddress = "nft.yearofchef.near"  # Year of chef 合约地址


blockchainLoader = MintbaseDocumentLoader(
    contract_address=contractAddress, blockchain_type="mainnet", api_key="omni-site"
)

nfts = blockchainLoader.load()

print(nfts[:1])

for doc in blockchainLoader.lazy_load():
    print()
    print(type(doc))
    print(doc)
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)