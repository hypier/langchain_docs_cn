---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/connery.ipynb
---
# Connery Action Tool

Using this tool, you can integrate individual Connery Action into your LangChain agent.

If you want to use more than one Connery Action in your agent,
check out the [Connery Toolkit](/docs/integrations/toolkits/connery) documentation.

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

## Example of using Connery Action Tool

In the example below, we fetch action by its ID from the Connery Runner and then call it with the specified parameters.

Here, we use the ID of the **Send email** action from the [Gmail](https://github.com/connery-io/gmail) plugin.


```python
%pip install -upgrade --quiet langchain-community
```


```python
import os

from langchain.agents import AgentType, initialize_agent
from langchain_community.tools.connery import ConneryService
from langchain_openai import ChatOpenAI

# Specify your Connery Runner credentials.
os.environ["CONNERY_RUNNER_URL"] = ""
os.environ["CONNERY_RUNNER_API_KEY"] = ""

# Specify OpenAI API key.
os.environ["OPENAI_API_KEY"] = ""

# Specify your email address to receive the emails from examples below.
recepient_email = "test@example.com"

# Get the SendEmail action from the Connery Runner by ID.
connery_service = ConneryService()
send_email_action = connery_service.get_action("CABC80BB79C15067CA983495324AE709")
```

Run the action manually.


```python
manual_run_result = send_email_action.run(
    {
        "recipient": recepient_email,
        "subject": "Test email",
        "body": "This is a test email sent from Connery.",
    }
)
print(manual_run_result)
```

Run the action using the OpenAI Functions agent.

You can see a LangSmith trace of this example [here](https://smith.langchain.com/public/a37d216f-c121-46da-a428-0e09dc19b1dc/r).


```python
llm = ChatOpenAI(temperature=0)
agent = initialize_agent(
    [send_email_action], llm, AgentType.OPENAI_FUNCTIONS, verbose=True
)
agent_run_result = agent.run(
    f"Send an email to the {recepient_email} and say that I will be late for the meeting."
)
print(agent_run_result)
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m
Invoking: `CABC80BB79C15067CA983495324AE709` with `{'recipient': 'test@example.com', 'subject': 'Late for Meeting', 'body': 'Dear Team,\n\nI wanted to inform you that I will be late for the meeting today. I apologize for any inconvenience caused. Please proceed with the meeting without me and I will join as soon as I can.\n\nBest regards,\n[Your Name]'}`


[0m[36;1m[1;3m{'messageId': '<d34a694d-50e0-3988-25da-e86b4c51d7a7@gmail.com>'}[0m[32;1m[1;3mI have sent an email to test@example.com informing them that you will be late for the meeting.[0m

[1m> Finished chain.[0m
I have sent an email to test@example.com informing them that you will be late for the meeting.
```
NOTE: Connery Action is a structured tool, so you can only use it in the agents supporting structured tools.


## Related

- Tool [conceptual guide](/docs/concepts/#tools)
- Tool [how-to guides](/docs/how_to/#tools)
