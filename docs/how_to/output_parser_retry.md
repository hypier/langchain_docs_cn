---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/output_parser_retry.ipynb
---

# 当解析错误发生时如何重试

虽然在某些情况下，仅通过查看输出可以修复任何解析错误，但在其他情况下则不行。一个例子是当输出不仅格式不正确，而且部分完整时。考虑下面的例子。

```python
from langchain.output_parsers import OutputFixingParser
from langchain_core.output_parsers import PydanticOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_openai import ChatOpenAI, OpenAI
```

```python
template = """根据用户的问题，提供一个操作和操作输入以说明应采取的步骤。
{format_instructions}
问题: {query}
响应:"""

class Action(BaseModel):
    action: str = Field(description="要采取的操作")
    action_input: str = Field(description="操作的输入")

parser = PydanticOutputParser(pydantic_object=Action)
```

```python
prompt = PromptTemplate(
    template="回答用户查询。\n{format_instructions}\n{query}\n",
    input_variables=["query"],
    partial_variables={"format_instructions": parser.get_format_instructions()},
)
```

```python
prompt_value = prompt.format_prompt(query="谁是莱昂纳多·迪卡普里奥的女友？")
```

```python
bad_response = '{"action": "search"}'
```

如果我们尝试直接解析这个响应，将会得到一个错误：

```python
parser.parse(bad_response)
```

```output
---------------------------------------------------------------------------
``````output
ValidationError                           Traceback (most recent call last)
``````output
File ~/workplace/langchain/libs/langchain/langchain/output_parsers/pydantic.py:30, in PydanticOutputParser.parse(self, text)
     29     json_object = json.loads(json_str, strict=False)
---> 30     return self.pydantic_object.parse_obj(json_object)
     32 except (json.JSONDecodeError, ValidationError) as e:
``````output
File ~/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/pydantic/main.py:526, in pydantic.main.BaseModel.parse_obj()
``````output
File ~/.pyenv/versions/3.10.1/envs/langchain/lib/python3.10/site-packages/pydantic/main.py:341, in pydantic.main.BaseModel.__init__()
``````output
ValidationError: 1 validation error for Action
action_input
  field required (type=value_error.missing)
``````output

在处理上述异常期间，发生了另一个异常：
``````output
OutputParserException                     Traceback (most recent call last)
``````output
Cell In[6], line 1
----> 1 parser.parse(bad_response)
``````output
File ~/workplace/langchain/libs/langchain/output_parsers/pydantic.py:35, in PydanticOutputParser.parse(self, text)
     33 name = self.pydantic_object.__name__
     34 msg = f"Failed to parse {name} from completion {text}. Got: {e}"
---> 35 raise OutputParserException(msg, llm_output=text)
``````output
OutputParserException: 无法从完成 {"action": "search"} 中解析 Action。得到：1 validation error for Action
action_input
  field required (type=value_error.missing)
```

如果我们尝试使用 `OutputFixingParser` 来修复这个错误，它会感到困惑——即，它不知道实际应该为操作输入放置什么。

```python
fix_parser = OutputFixingParser.from_llm(parser=parser, llm=ChatOpenAI())
```

```python
fix_parser.parse(bad_response)
```

```output
Action(action='search', action_input='input')
```

相反，我们可以使用 RetryOutputParser，它将提示（以及原始输出）传入以再次尝试获得更好的响应。

```python
from langchain.output_parsers import RetryOutputParser
```

```python
retry_parser = RetryOutputParser.from_llm(parser=parser, llm=OpenAI(temperature=0))
```

```python
retry_parser.parse_with_prompt(bad_response, prompt_value)
```

```output
Action(action='search', action_input='莱昂纳多·迪卡普里奥的女友')
```

我们还可以通过自定义链轻松添加 RetryOutputParser，将原始 LLM/ChatModel 输出转换为更可操作的格式。

```python
from langchain_core.runnables import RunnableLambda, RunnableParallel

completion_chain = prompt | OpenAI(temperature=0)

main_chain = RunnableParallel(
    completion=completion_chain, prompt_value=prompt
) | RunnableLambda(lambda x: retry_parser.parse_with_prompt(**x))

main_chain.invoke({"query": "谁是莱昂纳多·迪卡普里奥的女友？"})
```
```output
Action(action='search', action_input='莱昂纳多·迪卡普里奥的女友')
```
查找 [RetryOutputParser](https://api.python.langchain.com/en/latest/output_parsers/langchain.output_parsers.retry.RetryOutputParser.html#langchain.output_parsers.retry.RetryOutputParser) 的 API 文档。