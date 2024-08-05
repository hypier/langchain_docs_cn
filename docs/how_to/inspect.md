---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/inspect.ipynb
---

# 如何检查可运行对象

:::info 前提条件

本指南假设您对以下概念有一定了解：
- [LangChain 表达语言 (LCEL)](/docs/concepts/#langchain-expression-language)
- [链式可运行对象](/docs/how_to/sequence/)

:::

一旦您使用 [LangChain 表达语言](/docs/concepts/#langchain-expression-language) 创建了一个可运行对象，您可能会希望检查它，以更好地了解发生了什么。此笔记本涵盖了一些方法来实现这一点。

本指南展示了您可以以编程方式检查链的内部步骤的一些方法。如果您更感兴趣的是调试链中的问题，请参见[本节](/docs/how_to/debugging)。

首先，让我们创建一个示例链。我们将创建一个进行检索的链：

```python
%pip install -qU langchain langchain-openai faiss-cpu tiktoken
```

```python
from langchain_community.vectorstores import FAISS
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI, OpenAIEmbeddings

vectorstore = FAISS.from_texts(
    ["harrison worked at kensho"], embedding=OpenAIEmbeddings()
)
retriever = vectorstore.as_retriever()

template = """Answer the question based only on the following context:
{context}

Question: {question}
"""
prompt = ChatPromptTemplate.from_template(template)

model = ChatOpenAI()

chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)
```

## 获取图形

您可以使用 `get_graph()` 方法获取可运行的图形表示：


```python
chain.get_graph()
```

## 打印图形

虽然这不是很清晰，但你可以使用 `print_ascii()` 方法以更易于理解的方式显示该图形：

```python
chain.get_graph().print_ascii()
```
```output
           +---------------------------------+         
           | Parallel<context,question>Input |         
           +---------------------------------+         
                    **               **                
                 ***                   ***             
               **                         **           
+----------------------+              +-------------+  
| VectorStoreRetriever |              | Passthrough |  
+----------------------+              +-------------+  
                    **               **                
                      ***         ***                  
                         **     **                     
           +----------------------------------+        
           | Parallel<context,question>Output |        
           +----------------------------------+        
                             *                         
                             *                         
                             *                         
                  +--------------------+               
                  | ChatPromptTemplate |               
                  +--------------------+               
                             *                         
                             *                         
                             *                         
                      +------------+                   
                      | ChatOpenAI |                   
                      +------------+                   
                             *                         
                             *                         
                             *                         
                   +-----------------+                 
                   | StrOutputParser |                 
                   +-----------------+                 
                             *                         
                             *                         
                             *                         
                +-----------------------+              
                | StrOutputParserOutput |              
                +-----------------------+
```

## 获取提示

您可能想查看与 `get_prompts()` 方法一起使用的提示：


```python
chain.get_prompts()
```



```output
[ChatPromptTemplate(input_variables=['context', 'question'], messages=[HumanMessagePromptTemplate(prompt=PromptTemplate(input_variables=['context', 'question'], template='Answer the question based only on the following context:\n{context}\n\nQuestion: {question}\n'))])]
```

## 下一步

您现在已经学习了如何检查您编写的 LCEL 链。

接下来，请查看本节中关于可运行项的其他操作指南，或查看与 [调试您的链](/docs/how_to/debugging) 相关的操作指南。