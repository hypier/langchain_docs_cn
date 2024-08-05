---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/human_tools.ipynb
---

# 人类作为工具

人类是AGI，因此他们当然可以作为工具来帮助AI代理在困惑时提供帮助。

```python
%pip install --upgrade --quiet  langchain-community
```

```python
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import ChatOpenAI, OpenAI

llm = ChatOpenAI(temperature=0.0)
math_llm = OpenAI(temperature=0.0)
tools = load_tools(
    ["human", "llm-math"],
    llm=math_llm,
)

agent_chain = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
)
```

在上面的代码中，你可以看到工具直接从命令行获取输入。你可以根据需要自定义 `prompt_func` 和 `input_func`（如下所示）。

```python
agent_chain.run("What's my friend Eric's surname?")
# Answer with 'Zhu'
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3mI don't know Eric's surname, so I should ask a human for guidance.
Action: Human
Action Input: "What is Eric's surname?"[0m

What is Eric's surname?
``````output
 Zhu
``````output

Observation: [36;1m[1;3mZhu[0m
Thought:[32;1m[1;3mI now know Eric's surname is Zhu.
Final Answer: Eric's surname is Zhu.[0m

[1m> Finished chain.[0m
```


```output
"Eric's surname is Zhu."
```

## 配置输入函数

默认情况下，`HumanInputRun` 工具使用 Python 的 `input` 函数从用户获取输入。  
您可以将 input_func 自定义为您想要的任何内容。  
例如，如果您想接受多行输入，可以执行以下操作：


```python
def get_input() -> str:
    print("插入您的文本。输入 'q' 或按 Ctrl-D（在 Windows 上为 Ctrl-Z）以结束。")
    contents = []
    while True:
        try:
            line = input()
        except EOFError:
            break
        if line == "q":
            break
        contents.append(line)
    return "\n".join(contents)


# 您可以在加载时修改工具
tools = load_tools(["human", "ddg-search"], llm=math_llm, input_func=get_input)
```


```python
# 或者您可以直接实例化工具
from langchain_community.tools import HumanInputRun

tool = HumanInputRun(input_func=get_input)
```


```python
agent_chain = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
)
```


```python
agent_chain.run("我需要帮助归属一个引用")
```
```output


[1m> 进入新的 AgentExecutor 链...[0m
[32;1m[1;3m我应该向人类寻求指导
Action: Human
Action Input: "你能帮我归属一个引用吗？"[0m

你能帮我归属一个引用吗？
插入您的文本。输入 'q' 或按 Ctrl-D（在 Windows 上为 Ctrl-Z）以结束。
``````output
 vini
 vidi
 vici
 q
``````output

观察: [36;1m[1;3mvini
vidi
vici[0m
思考: [32;1m[1;3m我需要提供更多关于引用的上下文
Action: Human
Action Input: "这个引用是 'Veni, vidi, vici'"[0m

这个引用是 'Veni, vidi, vici'
插入您的文本。输入 'q' 或按 Ctrl-D（在 Windows 上为 Ctrl-Z）以结束。
``````output
 oh 谁说的 
 q
``````output

观察: [36;1m[1;3m哦，谁说的 [0m
思考: [32;1m[1;3m我可以使用 DuckDuckGo 搜索来找出是谁说的这个引用
Action: DuckDuckGo Search
Action Input: "谁说过 'Veni, vidi, vici'?"[0m
观察: [33;1m[1;3m更新于 2019 年 9 月 06 日。"Veni, vidi, vici" 是一个著名的短语，传说是罗马皇帝尤利乌斯·凯撒（公元前100-44年）在夸耀时说的，这让他那个时代及以后的许多作家印象深刻。这个短语大致意味着 "我来了，我看见了，我征服了"，可以发音为大约 Vehnee, Veedee ... Veni, vidi, vici（古典拉丁语：[weːniː wiːdiː wiːkiː]，教会拉丁语：[ˈveni ˈvidi ˈvitʃi]；"我来了；我看见了；我征服了"）是一个拉丁短语，用于指代迅速、决定性的胜利。这个短语通常归因于尤利乌斯·凯撒，根据阿皮安的说法，他在公元前47年以快速胜利后向罗马参议院写信时使用了这个短语 ... veni, vidi, vici 是尤利乌斯·凯撒的拉丁名言 ve· ni, vi· di, vi· ci ˌwā-nē ˌwē-dē ˈwē-kē ˌvā-nē ˌvē-dē ˈvē-chē : 我来了，我看见了，我征服了 与 'Veni, vidi, vici' 相关的文章 'In Vino Veritas' 和其他拉丁语... 词典条目靠近 veni, vidi, vici Venite veni, vidi, vici Venizélos 查看更多附近条目 引用此条目 风格 对于为什么 veni, vidi, vici 是一个流行说法，最简单的解释是它来自于历史上最著名的人物之一尤利乌斯·凯撒，并且有一个简单而强大的含义：我强大而迅速。但不仅仅是意义使这个短语如此强大。凯撒是一位天才作家，这个短语利用了拉丁语法 ... 作为最著名和最常被引用的拉丁表达之一，veni, vidi, vici 在几个世纪以来的数百次使用中作为胜利的表达。传说凯撒在享受胜利时使用了这些话。[0m
思考: [32;1m[1;3m我现在知道最终答案
最终答案: 尤利乌斯·凯撒说过 "Veni, vidi, vici" 这个引用，意思是 "我来了，我看见了，我征服了"。[0m

[1m> 完成链。[0m
```


```output
'尤利乌斯·凯撒说过 "Veni, vidi, vici" 这个引用，意思是 "我来了，我看见了，我征服了"。'
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)