---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/tensorflow_datasets.ipynb
---

# TensorFlow Datasets

>[TensorFlow Datasets](https://www.tensorflow.org/datasets) 是一个准备好使用的数据集集合，适用于 TensorFlow 或其他 Python 机器学习框架，如 Jax。所有数据集都以 [tf.data.Datasets](https://www.tensorflow.org/api_docs/python/tf/data/Dataset) 的形式提供，使得输入管道易于使用且性能优越。要开始使用，请参阅 [指南](https://www.tensorflow.org/datasets/overview) 和 [数据集列表](https://www.tensorflow.org/datasets/catalog/overview#all_datasets)。

该笔记本展示了如何将 `TensorFlow Datasets` 加载到我们可以在后续使用的文档格式中。

## 安装

您需要安装 `tensorflow` 和 `tensorflow-datasets` Python 包。

```python
%pip install --upgrade --quiet  tensorflow
```

```python
%pip install --upgrade --quiet  tensorflow-datasets
```

## 示例

作为示例，我们使用 [`mlqa/en` 数据集](https://www.tensorflow.org/datasets/catalog/mlqa#mlqaen)。

>`MLQA` (`Multilingual Question Answering Dataset`) 是一个用于评估多语言问答性能的基准数据集。该数据集包含 7 种语言：阿拉伯语、德语、西班牙语、英语、印地语、越南语和中文。
>
>- 主页: https://github.com/facebookresearch/MLQA
>- 源代码: `tfds.datasets.mlqa.Builder`
>- 下载大小: 72.21 MiB



```python
# `mlqa/en` 数据集的特征结构：

FeaturesDict(
    {
        "answers": Sequence(
            {
                "answer_start": int32,
                "text": Text(shape=(), dtype=string),
            }
        ),
        "context": Text(shape=(), dtype=string),
        "id": string,
        "question": Text(shape=(), dtype=string),
        "title": Text(shape=(), dtype=string),
    }
)
```


```python
import tensorflow as tf
import tensorflow_datasets as tfds
```


```python
# 尝试直接访问该数据集：
ds = tfds.load("mlqa/en", split="test")
ds = ds.take(1)  # 仅获取一个示例
ds
```



```output
<_TakeDataset element_spec={'answers': {'answer_start': TensorSpec(shape=(None,), dtype=tf.int32, name=None), 'text': TensorSpec(shape=(None,), dtype=tf.string, name=None)}, 'context': TensorSpec(shape=(), dtype=tf.string, name=None), 'id': TensorSpec(shape=(), dtype=tf.string, name=None), 'question': TensorSpec(shape=(), dtype=tf.string, name=None), 'title': TensorSpec(shape=(), dtype=tf.string, name=None)}>
```


现在我们需要创建一个自定义函数将数据集样本转换为文档。

这是一个要求。由于 TF 数据集没有标准格式，因此我们需要制作一个自定义转换函数。

我们将 `context` 字段用作 `Document.page_content`，并将其他字段放入 `Document.metadata`。



```python
from langchain_core.documents import Document


def decode_to_str(item: tf.Tensor) -> str:
    return item.numpy().decode("utf-8")


def mlqaen_example_to_document(example: dict) -> Document:
    return Document(
        page_content=decode_to_str(example["context"]),
        metadata={
            "id": decode_to_str(example["id"]),
            "title": decode_to_str(example["title"]),
            "question": decode_to_str(example["question"]),
            "answer": decode_to_str(example["answers"]["text"][0]),
        },
    )


for example in ds:
    doc = mlqaen_example_to_document(example)
    print(doc)
    break
```
```output
page_content='在 2006 年 2 月 23 日，女王玛丽 2号完成了环绕南美的旅程，遇见了她的同名船只，原始的 RMS 女王玛丽号，该船永久停靠在加利福尼亚州长滩。 在一队小船的护航下，两艘女王号交换了一个“鸣笛致敬”，整个长滩市都能听到。女王玛丽 2 号于 2008 年 1 月 13 日在纽约市港口的自由女神像附近与其他在役的库纳德邮轮女王维多利亚号和女王伊丽莎白 2号会面，并举行了庆祝烟火表演；女王伊丽莎白 2号和女王维多利亚号为这次会面进行了联合横渡大西洋。这标志着三艘库纳德女王号首次在同一地点出现。库纳德表示，这将是这三艘船最后一次相遇，因为女王伊丽莎白 2号将在 2008 年底退役。然而，事实并非如此，因为三艘女王号于 2008 年 4 月 22 日在南安普顿相遇。女王玛丽 2号于 2009 年 3 月 21 日星期六在迪拜与女王伊丽莎白 2号会合，此时后者已退役，两艘船停泊在拉希德港。随着女王伊丽莎白 2号从库纳德舰队中退役并停靠在迪拜，女王玛丽 2号成为唯一仍在积极运营的客轮。' metadata={'id': '5116f7cccdbf614d60bcd23498274ffd7b1e4ec7', 'title': 'RMS 女王玛丽 2号', 'question': '女王玛丽 2号完成环绕南美的旅程是哪一年？', 'answer': '2006'}
``````output
2023-08-03 14:27:08.482983: W tensorflow/core/kernels/data/cache_dataset_ops.cc:854] 调用的迭代器未完全读取正在缓存的数据集。为了避免数据集的意外截断，将丢弃数据集的部分缓存内容。如果您有类似于 `dataset.cache().take(k).repeat()` 的输入管道，则可能会发生这种情况。您应该使用 `dataset.take(k).cache().repeat()` 代替。
```

```python
from langchain_community.document_loaders import TensorflowDatasetLoader
from langchain_core.documents import Document

loader = TensorflowDatasetLoader(
    dataset_name="mlqa/en",
    split_name="test",
    load_max_docs=3,
    sample_to_document_function=mlqaen_example_to_document,
)
```

`TensorflowDatasetLoader` 具有以下参数：
- `dataset_name`: 要加载的数据集名称
- `split_name`: 要加载的分割名称。默认为 "train"。
- `load_max_docs`: 加载文档的数量限制。默认为 100。
- `sample_to_document_function`: 将数据集样本转换为文档的函数



```python
docs = loader.load()
len(docs)
```
```output
2023-08-03 14:27:22.998964: W tensorflow/core/kernels/data/cache_dataset_ops.cc:854] 调用的迭代器未完全读取正在缓存的数据集。为了避免数据集的意外截断，将丢弃数据集的部分缓存内容。如果您有类似于 `dataset.cache().take(k).repeat()` 的输入管道，则可能会发生这种情况。您应该使用 `dataset.take(k).cache().repeat()` 代替。
```


```output
3
```



```python
docs[0].page_content
```



```output
'在 2006 年 2 月 23 日，女王玛丽 2号完成了环绕南美的旅程，遇见了她的同名船只，原始的 RMS 女王玛丽号，该船永久停靠在加利福尼亚州长滩。 在一队小船的护航下，两艘女王号交换了一个“鸣笛致敬”，整个长滩市都能听到。女王玛丽 2 号于 2008 年 1 月 13 日在纽约市港口的自由女神像附近与其他在役的库纳德邮轮女王维多利亚号和女王伊丽莎白 2号会面，并举行了庆祝烟火表演；女王伊丽莎白 2号和女王维多利亚号为这次会面进行了联合横渡大西洋。这标志着三艘库纳德女王号首次在同一地点出现。库纳德表示，这将是这三艘船最后一次相遇，因为女王伊丽莎白 2号将在 2008 年底退役。然而，事实并非如此，因为三艘女王号于 2008 年 4 月 22 日在南安普顿相遇。女王玛丽 2号于 2009 年 3 月 21 日星期六在迪拜与女王伊丽莎白 2号会合，此时后者已退役，两艘船停泊在拉希德港。随着女王伊丽莎白 2号从库纳德舰队中退役并停靠在迪拜，女王玛丽 2号成为唯一仍在积极运营的客轮。'
```



```python
docs[0].metadata
```



```output
{'id': '5116f7cccdbf614d60bcd23498274ffd7b1e4ec7',
 'title': 'RMS 女王玛丽 2号',
 'question': '女王玛丽 2号完成环绕南美的旅程是哪一年？',
 'answer': '2006'}
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)