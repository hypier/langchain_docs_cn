---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/connery.ipynb
---
# Connery Toolkit

Using this toolkit, you can integrate Connery Actions into your LangChain agent.

If you want to use only one particular Connery Action in your agent,
check out the [Connery Action Tool](/docs/integrations/tools/connery) documentation.

## What is Connery?

Connery is an open-source plugin infrastructure for AI.

With Connery, you can easily create a custom plugin with a set of actions and seamlessly integrate them into your LangChain agent.
Connery will take care of critical aspects such as runtime, authorization, secret management, access management, audit logs, and other vital features.

Furthermore, Connery, supported by our community, provides a diverse collection of ready-to-use open-source plugins for added convenience.

Learn more about Connery:

- GitHub: https://github.com/connery-io/connery
- Documentation: https://docs.connery.io

## Prerequisites

To use Connery Actions in your LangChain agent, you need to do some preparation:

1. Set up the Connery runner using the [Quickstart](https://docs.connery.io/docs/runner/quick-start/) guide.
2. Install all the plugins with the actions you want to use in your agent.
3. Set environment variables `CONNERY_RUNNER_URL` and `CONNERY_RUNNER_API_KEY` so the toolkit can communicate with the Connery Runner.

## Example of using Connery Toolkit

In the example below, we create an agent that uses two Connery Actions to summarize a public webpage and send the summary by email:

1. **Summarize public webpage** action from the [Summarization](https://github.com/connery-io/summarization-plugin) plugin.
2. **Send email** action from the [Gmail](https://github.com/connery-io/gmail) plugin.

You can see a LangSmith trace of this example [here](https://smith.langchain.com/public/4af5385a-afe9-46f6-8a53-57fe2d63c5bc/r).


```python
%pip install -qU langchain-community
```


```python
import os

from langchain.agents import AgentType, initialize_agent
from langchain_community.agent_toolkits.connery import ConneryToolkit
from langchain_community.tools.connery import ConneryService
from langchain_openai import ChatOpenAI

# Specify your Connery Runner credentials.
os.environ["CONNERY_RUNNER_URL"] = ""
os.environ["CONNERY_RUNNER_API_KEY"] = ""

# Specify OpenAI API key.
os.environ["OPENAI_API_KEY"] = ""

# Specify your email address to receive the email with the summary from example below.
recepient_email = "test@example.com"

# Create a Connery Toolkit with all the available actions from the Connery Runner.
connery_service = ConneryService()
connery_toolkit = ConneryToolkit.create_instance(connery_service)

# Use OpenAI Functions agent to execute the prompt using actions from the Connery Toolkit.
llm = ChatOpenAI(temperature=0)
agent = initialize_agent(
    connery_toolkit.get_tools(), llm, AgentType.OPENAI_FUNCTIONS, verbose=True
)
result = agent.run(
    f"""Make a short summary of the webpage http://www.paulgraham.com/vb.html in three sentences
and send it to {recepient_email}. Include the link to the webpage into the body of the email."""
)
print(result)
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m
Invoking: `CA72DFB0AB4DF6C830B43E14B0782F70` with `{'publicWebpageUrl': 'http://www.paulgraham.com/vb.html'}`


[0m[33;1m[1;3m{'summary': 'The author reflects on the concept of life being short and how having children made them realize the true brevity of life. They discuss how time can be converted into discrete quantities and how limited certain experiences are. The author emphasizes the importance of prioritizing and eliminating unnecessary things in life, as well as actively pursuing meaningful experiences. They also discuss the negative impact of getting caught up in online arguments and the need to be aware of how time is being spent. The author suggests pruning unnecessary activities, not waiting to do things that matter, and savoring the time one has.'}[0m[32;1m[1;3m
Invoking: `CABC80BB79C15067CA983495324AE709` with `{'recipient': 'test@example.com', 'subject': 'Summary of the webpage', 'body': 'Here is a short summary of the webpage http://www.paulgraham.com/vb.html:\n\nThe author reflects on the concept of life being short and how having children made them realize the true brevity of life. They discuss how time can be converted into discrete quantities and how limited certain experiences are. The author emphasizes the importance of prioritizing and eliminating unnecessary things in life, as well as actively pursuing meaningful experiences. They also discuss the negative impact of getting caught up in online arguments and the need to be aware of how time is being spent. The author suggests pruning unnecessary activities, not waiting to do things that matter, and savoring the time one has.\n\nYou can find the full webpage [here](http://www.paulgraham.com/vb.html).'}`


[0m[33;1m[1;3m{'messageId': '<2f04b00e-122d-c7de-c91e-e78e0c3276d6@gmail.com>'}[0m[32;1m[1;3mI have sent the email with the summary of the webpage to test@example.com. Please check your inbox.[0m

[1m> Finished chain.[0m
I have sent the email with the summary of the webpage to test@example.com. Please check your inbox.
```
NOTE: Connery Action is a structured tool, so you can only use it in the agents supporting structured tools.
