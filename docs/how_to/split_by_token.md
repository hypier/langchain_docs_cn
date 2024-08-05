---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/split_by_token.ipynb
---

# 如何按标记拆分文本

语言模型有一个标记限制。您不应超过该标记限制。因此，当您将文本拆分为块时，计算标记数量是一个好主意。有许多标记器。在计算文本中的标记时，您应使用与语言模型中使用的相同标记器。

## tiktoken

:::note
[tiktoken](https://github.com/openai/tiktoken) 是由 `OpenAI` 创建的快速 `BPE` 分词器。
:::


我们可以使用 `tiktoken` 来估算使用的 tokens。它对 OpenAI 模型的估算可能会更准确。

1. 文本的拆分方式：通过传入的字符进行拆分。
2. 块大小的测量方式：由 `tiktoken` 分词器进行测量。

可以直接使用 [CharacterTextSplitter](https://api.python.langchain.com/en/latest/character/langchain_text_splitters.character.CharacterTextSplitter.html)、[RecursiveCharacterTextSplitter](https://api.python.langchain.com/en/latest/character/langchain_text_splitters.character.RecursiveCharacterTextSplitter.html) 和 [TokenTextSplitter](https://api.python.langchain.com/en/latest/base/langchain_text_splitters.base.TokenTextSplitter.html) 与 `tiktoken`。

```python
%pip install --upgrade --quiet langchain-text-splitters tiktoken
```


```python
from langchain_text_splitters import CharacterTextSplitter

# 这是一个可以拆分的长文档。
with open("state_of_the_union.txt") as f:
    state_of_the_union = f.read()
```

要使用 [CharacterTextSplitter](https://api.python.langchain.com/en/latest/character/langchain_text_splitters.character.CharacterTextSplitter.html) 进行拆分，然后使用 `tiktoken` 合并块，可以使用其 `.from_tiktoken_encoder()` 方法。请注意，该方法的拆分可能会大于 `tiktoken` 分词器测量的块大小。

`.from_tiktoken_encoder()` 方法接受 `encoding_name` 作为参数（例如 `cl100k_base`），或 `model_name`（例如 `gpt-4`）。所有其他参数如 `chunk_size`、`chunk_overlap` 和 `separators` 用于实例化 `CharacterTextSplitter`：


```python
text_splitter = CharacterTextSplitter.from_tiktoken_encoder(
    encoding_name="cl100k_base", chunk_size=100, chunk_overlap=0
)
texts = text_splitter.split_text(state_of_the_union)
```


```python
print(texts[0])
```
```output
Madam Speaker, Madam Vice President, our First Lady and Second Gentleman. Members of Congress and the Cabinet. Justices of the Supreme Court. My fellow Americans.  

Last year COVID-19 kept us apart. This year we are finally together again. 

Tonight, we meet as Democrats Republicans and Independents. But most importantly as Americans. 

With a duty to one another to the American people to the Constitution.
```
要对块大小实施严格限制，我们可以使用 `RecursiveCharacterTextSplitter.from_tiktoken_encoder`，其中每个拆分如果块大小较大，将递归拆分：


```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter.from_tiktoken_encoder(
    model_name="gpt-4",
    chunk_size=100,
    chunk_overlap=0,
)
```

我们还可以加载一个 `TokenTextSplitter` 分词器，它直接与 `tiktoken` 一起工作，并确保每个拆分都小于块大小。


```python
from langchain_text_splitters import TokenTextSplitter

text_splitter = TokenTextSplitter(chunk_size=10, chunk_overlap=0)

texts = text_splitter.split_text(state_of_the_union)
print(texts[0])
```
```output
Madam Speaker, Madam Vice President, our
```
某些书写语言（例如中文和日文）的字符编码为 2 个或更多的 tokens。直接使用 `TokenTextSplitter` 可能会导致在两个块之间拆分字符的 tokens，从而导致 Unicode 字符无效。使用 `RecursiveCharacterTextSplitter.from_tiktoken_encoder` 或 `CharacterTextSplitter.from_tiktoken_encoder` 以确保块包含有效的 Unicode 字符串。

## spaCy

:::note
[spaCy](https://spacy.io/) 是一个开源软件库，用于高级自然语言处理，使用编程语言 Python 和 Cython 编写。
:::

LangChain 实现了基于 [spaCy tokenizer](https://spacy.io/api/tokenizer) 的分割器。

1. 文本的分割方式：通过 `spaCy` tokenizer。
2. 块大小的测量方式：通过字符数。

```python
%pip install --upgrade --quiet  spacy
```

```python
# This is a long document we can split up.
with open("state_of_the_union.txt") as f:
    state_of_the_union = f.read()
```

```python
from langchain_text_splitters import SpacyTextSplitter

text_splitter = SpacyTextSplitter(chunk_size=1000)

texts = text_splitter.split_text(state_of_the_union)
print(texts[0])
```
```output
Madam Speaker, Madam Vice President, our First Lady and Second Gentleman.

Members of Congress and the Cabinet.

Justices of the Supreme Court.

My fellow Americans.  



Last year COVID-19 kept us apart.

This year we are finally together again. 



Tonight, we meet as Democrats Republicans and Independents.

But most importantly as Americans. 



With a duty to one another to the American people to the Constitution. 



And with an unwavering resolve that freedom will always triumph over tyranny. 



Six days ago, Russia’s Vladimir Putin sought to shake the foundations of the free world thinking he could make it bend to his menacing ways.

But he badly miscalculated. 



He thought he could roll into Ukraine and the world would roll over.

Instead he met a wall of strength he never imagined. 



He met the Ukrainian people. 



From President Zelenskyy to every Ukrainian, their fearlessness, their courage, their determination, inspires the world.
```

## SentenceTransformers

[SentenceTransformersTokenTextSplitter](https://api.python.langchain.com/en/latest/sentence_transformers/langchain_text_splitters.sentence_transformers.SentenceTransformersTokenTextSplitter.html) 是一个专门用于句子转换器模型的文本分割器。默认行为是将文本分割成适合您希望使用的句子转换器模型的令牌窗口的块。

要根据句子转换器的分词器分割文本并限制令牌数量，请实例化一个 `SentenceTransformersTokenTextSplitter`。您可以选择性地指定：

- `chunk_overlap`: 令牌重叠的整数计数；
- `model_name`: 句子转换器模型名称，默认为 `"sentence-transformers/all-mpnet-base-v2"`；
- `tokens_per_chunk`: 每个块所需的令牌数量。

```python
from langchain_text_splitters import SentenceTransformersTokenTextSplitter

splitter = SentenceTransformersTokenTextSplitter(chunk_overlap=0)
text = "Lorem "

count_start_and_stop_tokens = 2
text_token_count = splitter.count_tokens(text=text) - count_start_and_stop_tokens
print(text_token_count)
```
```output
2
```

```python
token_multiplier = splitter.maximum_tokens_per_chunk // text_token_count + 1

# `text_to_split` does not fit in a single chunk
text_to_split = text * token_multiplier

print(f"tokens in text to split: {splitter.count_tokens(text=text_to_split)}")
```
```output
tokens in text to split: 514
```

```python
text_chunks = splitter.split_text(text=text_to_split)

print(text_chunks[1])
```
```output
lorem
```

## NLTK

:::note
[自然语言工具包](https://en.wikipedia.org/wiki/Natural_Language_Toolkit)，更常见的名称是 [NLTK](https://www.nltk.org/)，是一个用于符号和统计自然语言处理（NLP）的库和程序套件，专为用Python编程语言编写的英语处理而设计。
:::


与其仅仅在 "\n\n" 处分割，我们可以使用 `NLTK` 基于 [NLTK 分词器](https://www.nltk.org/api/nltk.tokenize.html) 进行分割。

1. 文本的分割方式：通过 `NLTK` 分词器。
2. 块大小的测量方式：按字符数计算。


```python
# pip install nltk
```


```python
# This is a long document we can split up.
with open("state_of_the_union.txt") as f:
    state_of_the_union = f.read()
```


```python
from langchain_text_splitters import NLTKTextSplitter

text_splitter = NLTKTextSplitter(chunk_size=1000)
```


```python
texts = text_splitter.split_text(state_of_the_union)
print(texts[0])
```
```output
Madam Speaker, Madam Vice President, our First Lady and Second Gentleman.

Members of Congress and the Cabinet.

Justices of the Supreme Court.

My fellow Americans.

Last year COVID-19 kept us apart.

This year we are finally together again.

Tonight, we meet as Democrats Republicans and Independents.

But most importantly as Americans.

With a duty to one another to the American people to the Constitution.

And with an unwavering resolve that freedom will always triumph over tyranny.

Six days ago, Russia’s Vladimir Putin sought to shake the foundations of the free world thinking he could make it bend to his menacing ways.

But he badly miscalculated.

He thought he could roll into Ukraine and the world would roll over.

Instead he met a wall of strength he never imagined.

He met the Ukrainian people.

From President Zelenskyy to every Ukrainian, their fearlessness, their courage, their determination, inspires the world.

Groups of citizens blocking tanks with their bodies.
```

## KoNLPY

:::note
[KoNLPy: Korean NLP in Python](https://konlpy.org/en/latest/) 是一个用于韩语自然语言处理（NLP）的 Python 包。
:::

Token splitting 涉及将文本分割成更小、更易管理的单元，称为 tokens。这些 tokens 通常是单词、短语、符号或其他对进一步处理和分析至关重要的有意义元素。在像英语这样的语言中，token splitting 通常涉及通过空格和标点符号来分隔单词。token splitting 的有效性在很大程度上取决于分词器对语言结构的理解，以确保生成有意义的 tokens。由于为英语设计的分词器无法理解其他语言（如韩语）的独特语义结构，因此无法有效用于韩语处理。

### 使用 KoNLPY 的 Kkma 分析器进行韩文分词
对于韩文文本，KoNLPY 包含一个形态分析器，称为 `Kkma`（Korean Knowledge Morpheme Analyzer）。`Kkma` 提供韩文文本的详细形态分析。它将句子分解为单词，并将单词分解为各自的语素，识别每个词元的词性。它可以将一段文本分割成独立的句子，这对于处理长文本特别有用。

### 使用注意事项
虽然 `Kkma` 以其详细的分析而闻名，但需要注意的是，这种精确性可能会影响处理速度。因此，`Kkma` 最适合用于分析深度优先于快速文本处理的应用程序。

```python
# pip install konlpy
```

```python
# This is a long Korean document that we want to split up into its component sentences.
with open("./your_korean_doc.txt") as f:
    korean_document = f.read()
```

```python
from langchain_text_splitters import KonlpyTextSplitter

text_splitter = KonlpyTextSplitter()
```

```python
texts = text_splitter.split_text(korean_document)
# The sentences are split with "\n\n" characters.
print(texts[0])
```
```output
춘향전 옛날에 남원에 이 도령이라는 벼슬아치 아들이 있었다.

그의 외모는 빛나는 달처럼 잘생겼고, 그의 학식과 기예는 남보다 뛰어났다.

한편, 이 마을에는 춘향이라는 절세 가인이 살고 있었다.

춘 향의 아름다움은 꽃과 같아 마을 사람들 로부터 많은 사랑을 받았다.

어느 봄날, 도령은 친구들과 놀러 나갔다가 춘 향을 만 나 첫 눈에 반하고 말았다.

두 사람은 서로 사랑하게 되었고, 이내 비밀스러운 사랑의 맹세를 나누었다.

하지만 좋은 날들은 오래가지 않았다.

도령의 아버지가 다른 곳으로 전근을 가게 되어 도령도 떠나 야만 했다.

이별의 아픔 속에서도, 두 사람은 재회를 기약하며 서로를 믿고 기다리기로 했다.

그러나 새로 부임한 관아의 사또가 춘 향의 아름다움에 욕심을 내 어 그녀에게 강요를 시작했다.

춘 향 은 도령에 대한 자신의 사랑을 지키기 위해, 사또의 요구를 단호히 거절했다.

이에 분노한 사또는 춘 향을 감옥에 가두고 혹독한 형벌을 내렸다.

이야기는 이 도령이 고위 관직에 오른 후, 춘 향을 구해 내는 것으로 끝난다.

두 사람은 오랜 시련 끝에 다시 만나게 되고, 그들의 사랑은 온 세상에 전해 지며 후세에까지 이어진다.

- 춘향전 (The Tale of Chunhyang)
```

## Hugging Face 分词器

[Hugging Face](https://huggingface.co/docs/tokenizers/index) 有许多分词器。

我们使用 Hugging Face 分词器，即 [GPT2TokenizerFast](https://huggingface.co/Ransaka/gpt2-tokenizer-fast) 来计算文本的标记长度。

1. 文本如何拆分：通过传入的字符。
2. 块大小如何测量：通过 `Hugging Face` 分词器计算的标记数量。


```python
from transformers import GPT2TokenizerFast

tokenizer = GPT2TokenizerFast.from_pretrained("gpt2")
```


```python
# This is a long document we can split up.
with open("state_of_the_union.txt") as f:
    state_of_the_union = f.read()
from langchain_text_splitters import CharacterTextSplitter
```


```python
text_splitter = CharacterTextSplitter.from_huggingface_tokenizer(
    tokenizer, chunk_size=100, chunk_overlap=0
)
texts = text_splitter.split_text(state_of_the_union)
```


```python
print(texts[0])
```
```output
Madam Speaker, Madam Vice President, our First Lady and Second Gentleman. Members of Congress and the Cabinet. Justices of the Supreme Court. My fellow Americans.  

Last year COVID-19 kept us apart. This year we are finally together again. 

Tonight, we meet as Democrats Republicans and Independents. But most importantly as Americans. 

With a duty to one another to the American people to the Constitution.
```