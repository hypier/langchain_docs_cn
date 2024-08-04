---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/tools_error.ipynb
---
# How to handle tool errors

:::info Prerequisites

This guide assumes familiarity with the following concepts:
- [Chat models](/docs/concepts/#chat-models)
- [LangChain Tools](/docs/concepts/#tools)
- [How to use a model to call tools](/docs/how_to/tool_calling)

:::

Calling tools with an LLM is generally more reliable than pure prompting, but it isn't perfect. The model may try to call a tool that doesn't exist or fail to return arguments that match the requested schema. Strategies like keeping schemas simple, reducing the number of tools you pass at once, and having good names and descriptions can help mitigate this risk, but aren't foolproof.

This guide covers some ways to build error handling into your chains to mitigate these failure modes.

## Setup

We'll need to install the following packages:


```python
%pip install --upgrade --quiet langchain-core langchain-openai
```

If you'd like to trace your runs in [LangSmith](https://docs.smith.langchain.com/) uncomment and set the following environment variables:


```python
import getpass
import os

# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## Chain

Suppose we have the following (dummy) tool and tool-calling chain. We'll make our tool intentionally convoluted to try and trip up the model.

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm"/>


```python
# Define tool
from langchain_core.tools import tool


@tool
def complex_tool(int_arg: int, float_arg: float, dict_arg: dict) -> int:
    """Do something complex with a complex tool."""
    return int_arg * float_arg


llm_with_tools = llm.bind_tools(
    [complex_tool],
)

# Define chain
chain = llm_with_tools | (lambda msg: msg.tool_calls[0]["args"]) | complex_tool
```

We can see that when we try to invoke this chain with even a fairly explicit input, the model fails to correctly call the tool (it forgets the `dict_arg` argument).


```python
chain.invoke(
    "use complex tool. the args are 5, 2.1, empty dictionary. don't forget dict_arg"
)
```

```output
---------------------------------------------------------------------------
``````output
ValidationError                           Traceback (most recent call last)
``````output
Cell In[6], line 1
----> 1 chain.invoke(
      2     "use complex tool. the args are 5, 2.1, empty dictionary. don't forget dict_arg"
      3 )
``````output
File ~/.pyenv/versions/3.10.5/lib/python3.10/site-packages/langchain_core/runnables/base.py:2572, in RunnableSequence.invoke(self, input, config, **kwargs)
   2570             input = step.invoke(input, config, **kwargs)
   2571         else:
-> 2572             input = step.invoke(input, config)
   2573 # finish the root run
   2574 except BaseException as e:
``````output
File ~/.pyenv/versions/3.10.5/lib/python3.10/site-packages/langchain_core/tools.py:380, in BaseTool.invoke(self, input, config, **kwargs)
    373 def invoke(
    374     self,
    375     input: Union[str, Dict],
    376     config: Optional[RunnableConfig] = None,
    377     **kwargs: Any,
    378 ) -> Any:
    379     config = ensure_config(config)
--> 380     return self.run(
    381         input,
    382         callbacks=config.get("callbacks"),
    383         tags=config.get("tags"),
    384         metadata=config.get("metadata"),
    385         run_name=config.get("run_name"),
    386         run_id=config.pop("run_id", None),
    387         config=config,
    388         **kwargs,
    389     )
``````output
File ~/.pyenv/versions/3.10.5/lib/python3.10/site-packages/langchain_core/tools.py:537, in BaseTool.run(self, tool_input, verbose, start_color, color, callbacks, tags, metadata, run_name, run_id, config, **kwargs)
    535 except ValidationError as e:
    536     if not self.handle_validation_error:
--> 537         raise e
    538     elif isinstance(self.handle_validation_error, bool):
    539         observation = "Tool input validation error"
``````output
File ~/.pyenv/versions/3.10.5/lib/python3.10/site-packages/langchain_core/tools.py:526, in BaseTool.run(self, tool_input, verbose, start_color, color, callbacks, tags, metadata, run_name, run_id, config, **kwargs)
    524 context = copy_context()
    525 context.run(_set_config_context, child_config)
--> 526 parsed_input = self._parse_input(tool_input)
    527 tool_args, tool_kwargs = self._to_args_and_kwargs(parsed_input)
    528 observation = (
    529     context.run(
    530         self._run, *tool_args, run_manager=run_manager, **tool_kwargs
   (...)
    533     else context.run(self._run, *tool_args, **tool_kwargs)
    534 )
``````output
File ~/.pyenv/versions/3.10.5/lib/python3.10/site-packages/langchain_core/tools.py:424, in BaseTool._parse_input(self, tool_input)
    422 else:
    423     if input_args is not None:
--> 424         result = input_args.parse_obj(tool_input)
    425         return {
    426             k: getattr(result, k)
    427             for k, v in result.dict().items()
    428             if k in tool_input
    429         }
    430 return tool_input
``````output
File ~/.pyenv/versions/3.10.5/lib/python3.10/site-packages/pydantic/main.py:526, in pydantic.main.BaseModel.parse_obj()
``````output
File ~/.pyenv/versions/3.10.5/lib/python3.10/site-packages/pydantic/main.py:341, in pydantic.main.BaseModel.__init__()
``````output
ValidationError: 1 validation error for complex_toolSchema
dict_arg
  field required (type=value_error.missing)
```

## Try/except tool call

The simplest way to more gracefully handle errors is to try/except the tool-calling step and return a helpful message on errors:


```python
from typing import Any

from langchain_core.runnables import Runnable, RunnableConfig


def try_except_tool(tool_args: dict, config: RunnableConfig) -> Runnable:
    try:
        complex_tool.invoke(tool_args, config=config)
    except Exception as e:
        return f"Calling tool with arguments:\n\n{tool_args}\n\nraised the following error:\n\n{type(e)}: {e}"


chain = llm_with_tools | (lambda msg: msg.tool_calls[0]["args"]) | try_except_tool

print(
    chain.invoke(
        "use complex tool. the args are 5, 2.1, empty dictionary. don't forget dict_arg"
    )
)
```
```output
Calling tool with arguments:

{'int_arg': 5, 'float_arg': 2.1}

raised the following error:

<class 'pydantic.error_wrappers.ValidationError'>: 1 validation error for complex_toolSchema
dict_arg
  field required (type=value_error.missing)
```
## Fallbacks

We can also try to fallback to a better model in the event of a tool invocation error. In this case we'll fall back to an identical chain that uses `gpt-4-1106-preview` instead of `gpt-3.5-turbo`.


```python
chain = llm_with_tools | (lambda msg: msg.tool_calls[0]["args"]) | complex_tool

better_model = ChatOpenAI(model="gpt-4-1106-preview", temperature=0).bind_tools(
    [complex_tool], tool_choice="complex_tool"
)

better_chain = better_model | (lambda msg: msg.tool_calls[0]["args"]) | complex_tool

chain_with_fallback = chain.with_fallbacks([better_chain])

chain_with_fallback.invoke(
    "use complex tool. the args are 5, 2.1, empty dictionary. don't forget dict_arg"
)
```



```output
10.5
```


Looking at the [LangSmith trace](https://smith.langchain.com/public/00e91fc2-e1a4-4b0f-a82e-e6b3119d196c/r) for this chain run, we can see that the first chain call fails as expected and it's the fallback that succeeds.

## Retry with exception

To take things one step further, we can try to automatically re-run the chain with the exception passed in, so that the model may be able to correct its behavior:


```python
from langchain_core.messages import AIMessage, HumanMessage, ToolCall, ToolMessage
from langchain_core.prompts import ChatPromptTemplate


class CustomToolException(Exception):
    """Custom LangChain tool exception."""

    def __init__(self, tool_call: ToolCall, exception: Exception) -> None:
        super().__init__()
        self.tool_call = tool_call
        self.exception = exception


def tool_custom_exception(msg: AIMessage, config: RunnableConfig) -> Runnable:
    try:
        return complex_tool.invoke(msg.tool_calls[0]["args"], config=config)
    except Exception as e:
        raise CustomToolException(msg.tool_calls[0], e)


def exception_to_messages(inputs: dict) -> dict:
    exception = inputs.pop("exception")

    # Add historical messages to the original input, so the model knows that it made a mistake with the last tool call.
    messages = [
        AIMessage(content="", tool_calls=[exception.tool_call]),
        ToolMessage(
            tool_call_id=exception.tool_call["id"], content=str(exception.exception)
        ),
        HumanMessage(
            content="The last tool call raised an exception. Try calling the tool again with corrected arguments. Do not repeat mistakes."
        ),
    ]
    inputs["last_output"] = messages
    return inputs


# We add a last_output MessagesPlaceholder to our prompt which if not passed in doesn't
# affect the prompt at all, but gives us the option to insert an arbitrary list of Messages
# into the prompt if needed. We'll use this on retries to insert the error message.
prompt = ChatPromptTemplate.from_messages(
    [("human", "{input}"), ("placeholder", "{last_output}")]
)
chain = prompt | llm_with_tools | tool_custom_exception

# If the initial chain call fails, we rerun it withe the exception passed in as a message.
self_correcting_chain = chain.with_fallbacks(
    [exception_to_messages | chain], exception_key="exception"
)
```


```python
self_correcting_chain.invoke(
    {
        "input": "use complex tool. the args are 5, 2.1, empty dictionary. don't forget dict_arg"
    }
)
```



```output
10.5
```


And our chain succeeds! Looking at the [LangSmith trace](https://smith.langchain.com/public/c11e804c-e14f-4059-bd09-64766f999c14/r), we can see that indeed our initial chain still fails, and it's only on retrying that the chain succeeds.

## Next steps

Now you've seen some strategies how to handle tool calling errors. Next, you can learn more about how to use tools:

- Few shot prompting [with tools](/docs/how_to/tools_few_shot/)
- Stream [tool calls](/docs/how_to/tool_streaming/)
- Pass [runtime values to tools](/docs/how_to/tool_runtime)

You can also check out some more specific uses of tool calling:

- Getting [structured outputs](/docs/how_to/structured_output/) from models
