---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/blockchain.ipynb
---

# 区块链

## 概述

本笔记本的目的是提供一种测试 Langchain Document Loader 在区块链上功能的方法。

最初，此加载器支持：

*   从 NFT 智能合约（ERC721 和 ERC1155）加载 NFTs 作为文档
*   以太坊主网、以太坊测试网、Polygon 主网、Polygon 测试网（默认是 eth-mainnet）
*   Alchemy 的 getNFTsForCollection API

如果社区认为此加载器有价值，可以进行扩展。具体来说：

*   可以添加额外的 API（例如，与交易相关的 API）

此文档加载器需要：

*   一个免费的 [Alchemy API 密钥](https://www.alchemy.com/)

输出格式如下：

- pageContent= Individual NFT
- metadata={'source': '0x1a92f7381b9f03921564a437210bb9396471050c', 'blockchain': 'eth-mainnet', 'tokenId': '0x15'})

## 将NFT加载到文档加载器中


```python
# get ALCHEMY_API_KEY from https://www.alchemy.com/

alchemyApiKey = "..."
```

### 选项 1：以太坊主网（默认 BlockchainType）


```python
from langchain_community.document_loaders.blockchain import (
    BlockchainDocumentLoader,
    BlockchainType,
)

contractAddress = "0xbc4ca0eda7647a8ab7c2061c2e118a18a936f13d"  # Bored Ape Yacht Club 合约地址

blockchainType = BlockchainType.ETH_MAINNET  # 默认值，可选参数

blockchainLoader = BlockchainDocumentLoader(
    contract_address=contractAddress, api_key=alchemyApiKey
)

nfts = blockchainLoader.load()

nfts[:2]
```

### 选项 2：Polygon 主网


```python
contractAddress = (
    "0x448676ffCd0aDf2D85C1f0565e8dde6924A9A7D9"  # Polygon Mainnet contract address
)

blockchainType = BlockchainType.POLYGON_MAINNET

blockchainLoader = BlockchainDocumentLoader(
    contract_address=contractAddress,
    blockchainType=blockchainType,
    api_key=alchemyApiKey,
)

nfts = blockchainLoader.load()

nfts[:2]
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)