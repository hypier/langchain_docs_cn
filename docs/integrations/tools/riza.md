---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/riza.ipynb
---

# Riza 代码解释器

> Riza 代码解释器是一个基于WASM的隔离环境，用于运行由AI代理生成的Python或JavaScript。

在这个笔记本中，我们将创建一个使用Python解决LLM无法独立解决的问题的代理示例：
计算单词“strawberry”中'r'的数量。

在开始之前，从[Riza仪表板](https://dashboard.riza.io)获取一个API密钥。有关更多指南和完整的API参考，请访问[Riza代码解释器API文档](https://docs.riza.io)。

确保您已安装必要的依赖项。


```python
%pip install --upgrade --quiet langchain-community rizaio
```

将您的API密钥设置为环境变量。


```python
%env ANTHROPIC_API_KEY=<your_anthropic_api_key_here>
%env RIZA_API_KEY=<your_riza_api_key_here>
```


```python
from langchain_community.tools.riza.command import ExecPython
```


```python
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_anthropic import ChatAnthropic
from langchain_core.prompts import ChatPromptTemplate
```

初始化`ExecPython`工具。


```python
tools = [ExecPython()]
```

使用Anthropic的Claude Haiku模型初始化一个代理。


```python
llm = ChatAnthropic(model="claude-3-haiku-20240307", temperature=0)

prompt_template = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "你是一个有帮助的助手。如果需要解决问题，请确保使用工具。",
        ),
        ("human", "{input}"),
        ("placeholder", "{agent_scratchpad}"),
    ]
)

agent = create_tool_calling_agent(llm, tools, prompt_template)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```


```python
# 提出一个棘手的问题
result = agent_executor.invoke({"input": "how many rs are in strawberry?"})
print(result["output"][0]["text"])
```
```output


[1m> 进入新的AgentExecutor链...[0m
[32;1m[1;3m
调用: `riza_exec_python`，参数为 `{'code': 'word = "strawberry"\nprint(word.count("r"))'}`
回应: [{'id': 'toolu_01JwPLAAqqCNCjVuEnK8Fgut', 'input': {}, 'name': 'riza_exec_python', 'type': 'tool_use', 'index': 0, 'partial_json': '{"code": "word = \\"strawberry\\"\\nprint(word.count(\\"r\\"))"}'}]

[0m[36;1m[1;3m3
[0m[32;1m[1;3m[{'text': '\n\n单词 "strawberry" 包含 3 个 "r" 字符。', 'type': 'text', 'index': 0}][0m

[1m> 完成链。[0m


单词 "strawberry" 包含 3 个 "r" 字符。
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)