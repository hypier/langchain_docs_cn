---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/chatbots_tools.ipynb
---
# How to add tools to chatbots

:::info Prerequisites

This guide assumes familiarity with the following concepts:

- [Chatbots](/docs/concepts/#messages)
- [Agents](/docs/tutorials/agents)
- [Chat history](/docs/concepts/#chat-history)

:::

This section will cover how to create conversational agents: chatbots that can interact with other systems and APIs using tools.

## Setup

For this guide, we'll be using a [tool calling agent](/docs/how_to/agent_executor) with a single tool for searching the web. The default will be powered by [Tavily](/docs/integrations/tools/tavily_search), but you can switch it out for any similar tool. The rest of this section will assume you're using Tavily.

You'll need to [sign up for an account](https://tavily.com/) on the Tavily website, and install the following packages:


```python
%pip install --upgrade --quiet langchain-community langchain-openai tavily-python

# Set env var OPENAI_API_KEY or load from a .env file:
import dotenv

dotenv.load_dotenv()
```

You will also need your OpenAI key set as `OPENAI_API_KEY` and your Tavily API key set as `TAVILY_API_KEY`.

## Creating an agent

Our end goal is to create an agent that can respond conversationally to user questions while looking up information as needed.

First, let's initialize Tavily and an OpenAI chat model capable of tool calling:


```python
from langchain_community.tools.tavily_search import TavilySearchResults
from langchain_openai import ChatOpenAI

tools = [TavilySearchResults(max_results=1)]

# Choose the LLM that will drive the agent
# Only certain models support this
chat = ChatOpenAI(model="gpt-3.5-turbo-1106", temperature=0)
```

To make our agent conversational, we must also choose a prompt with a placeholder for our chat history. Here's an example:


```python
from langchain_core.prompts import ChatPromptTemplate

# Adapted from https://smith.langchain.com/hub/jacob/tool-calling-agent
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant. You may not need to use tools for every query - the user may just want to chat!",
        ),
        ("placeholder", "{messages}"),
        ("placeholder", "{agent_scratchpad}"),
    ]
)
```

Great! Now let's assemble our agent:


```python
from langchain.agents import AgentExecutor, create_tool_calling_agent

agent = create_tool_calling_agent(chat, tools, prompt)

agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```

## Running the agent

Now that we've set up our agent, let's try interacting with it! It can handle both trivial queries that require no lookup:


```python
from langchain_core.messages import HumanMessage

agent_executor.invoke({"messages": [HumanMessage(content="I'm Nemo!")]})
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3mHello Nemo! It's great to meet you. How can I assist you today?[0m

[1m> Finished chain.[0m
```


```output
{'messages': [HumanMessage(content="I'm Nemo!")],
 'output': "Hello Nemo! It's great to meet you. How can I assist you today?"}
```


Or, it can use of the passed search tool to get up to date information if needed:


```python
agent_executor.invoke(
    {
        "messages": [
            HumanMessage(
                content="What is the current conservation status of the Great Barrier Reef?"
            )
        ],
    }
)
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m
Invoking: `tavily_search_results_json` with `{'query': 'current conservation status of the Great Barrier Reef'}`


[0m[36;1m[1;3m[{'url': 'https://www.abc.net.au/news/2022-08-04/great-barrier-reef-report-says-coral-recovering-after-bleaching/101296186', 'content': 'Great Barrier Reef hit with widespread and severe bleaching event\n\'Devastating\': Over 90pc of reefs on Great Barrier Reef suffered bleaching over summer, report reveals\nTop Stories\nJailed Russian opposition leader Alexei Navalny is dead, says prison service\nTaylor Swift puts an Aussie twist on a classic as she packs the MCG for the biggest show of her career — as it happened\nMelbourne comes alive with Swifties, as even those without tickets turn up to soak in the atmosphere\nAustralian Border Force investigates after arrival of more than 20 men by boat north of Broome\nOpenAI launches video model that can instantly create short clips from text prompts\nAntoinette Lattouf loses bid to force ABC to produce emails calling for her dismissal\nCategory one cyclone makes landfall in Gulf of Carpentaria off NT-Queensland border\nWhy the RBA may be forced to cut before the Fed\nBrisbane records \'wettest day since 2022\', as woman dies in floodwaters near Mount Isa\n$45m Sydney beachside home once owned by late radio star is demolished less than a year after sale\nAnnabel Sutherland\'s historic double century puts Australia within reach of Test victory over South Africa\nAlmighty defensive effort delivers Indigenous victory in NRL All Stars clash\nLisa Wilkinson feared she would have to sell home to pay legal costs of Bruce Lehrmann\'s defamation case, court documents reveal\nSupermarkets as you know them are disappearing from our cities\nNRL issues Broncos\' Reynolds, Carrigan with breach notices after public scrap\nPopular Now\nJailed Russian opposition leader Alexei Navalny is dead, says prison service\nTaylor Swift puts an Aussie twist on a classic as she packs the MCG for the biggest show of her career — as it happened\n$45m Sydney beachside home once owned by late radio star is demolished less than a year after sale\nAustralian Border Force investigates after arrival of more than 20 men by boat north of Broome\nDealer sentenced for injecting children as young as 12 with methylamphetamine\nMelbourne comes alive with Swifties, as even those without tickets turn up to soak in the atmosphere\nTop Stories\nJailed Russian opposition leader Alexei Navalny is dead, says prison service\nTaylor Swift puts an Aussie twist on a classic as she packs the MCG for the biggest show of her career — as it happened\nMelbourne comes alive with Swifties, as even those without tickets turn up to soak in the atmosphere\nAustralian Border Force investigates after arrival of more than 20 men by boat north of Broome\nOpenAI launches video model that can instantly create short clips from text prompts\nJust In\nJailed Russian opposition leader Alexei Navalny is dead, says prison service\nMelbourne comes alive with Swifties, as even those without tickets turn up to soak in the atmosphere\nTraveller alert after one-year-old in Adelaide reported with measles\nAntoinette Lattouf loses bid to force ABC to produce emails calling for her dismissal\nFooter\nWe acknowledge Aboriginal and Torres Strait Islander peoples as the First Australians and Traditional Custodians of the lands where we live, learn, and work.\n Increased coral cover could come at a cost\nThe rapid growth in coral cover appears to have come at the expense of the diversity of coral on the reef, with most of the increases accounted for by fast-growing branching coral called Acropora.\n Documents obtained by the ABC under Freedom of Information laws revealed the Morrison government had forced AIMS to rush the report\'s release and orchestrated a "leak" of the material to select media outlets ahead of the reef being considered for inclusion on the World Heritage In Danger list.\n The reef\'s status and potential inclusion on the In Danger list were due to be discussed at the 45th session of the World Heritage Committee in Russia in June this year, but the meeting was indefinitely postponed due to the war in Ukraine.\n More from ABC\nEditorial Policies\nGreat Barrier Reef coral cover at record levels after mass-bleaching events, report shows\nGreat Barrier Reef coral cover at record levels after mass-bleaching events, report shows\nRecord coral cover is being seen across much of the Great Barrier Reef as it recovers from past storms and mass-bleaching events.'}][0m[32;1m[1;3mThe Great Barrier Reef is currently showing signs of recovery, with record coral cover being seen across much of the reef. This recovery comes after past storms and mass-bleaching events. However, the rapid growth in coral cover appears to have come at the expense of the diversity of coral on the reef, with most of the increases accounted for by fast-growing branching coral called Acropora. There were discussions about the reef's potential inclusion on the World Heritage In Danger list, but the meeting to consider this was indefinitely postponed due to the war in Ukraine.

You can read more about it in this article: [Great Barrier Reef hit with widespread and severe bleaching event](https://www.abc.net.au/news/2022-08-04/great-barrier-reef-report-says-coral-recovering-after-bleaching/101296186)[0m

[1m> Finished chain.[0m
```


```output
{'messages': [HumanMessage(content='What is the current conservation status of the Great Barrier Reef?')],
 'output': "The Great Barrier Reef is currently showing signs of recovery, with record coral cover being seen across much of the reef. This recovery comes after past storms and mass-bleaching events. However, the rapid growth in coral cover appears to have come at the expense of the diversity of coral on the reef, with most of the increases accounted for by fast-growing branching coral called Acropora. There were discussions about the reef's potential inclusion on the World Heritage In Danger list, but the meeting to consider this was indefinitely postponed due to the war in Ukraine.\n\nYou can read more about it in this article: [Great Barrier Reef hit with widespread and severe bleaching event](https://www.abc.net.au/news/2022-08-04/great-barrier-reef-report-says-coral-recovering-after-bleaching/101296186)"}
```


## Conversational responses

Because our prompt contains a placeholder for chat history messages, our agent can also take previous interactions into account and respond conversationally like a standard chatbot:


```python
from langchain_core.messages import AIMessage, HumanMessage

agent_executor.invoke(
    {
        "messages": [
            HumanMessage(content="I'm Nemo!"),
            AIMessage(content="Hello Nemo! How can I assist you today?"),
            HumanMessage(content="What is my name?"),
        ],
    }
)
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3mYour name is Nemo![0m

[1m> Finished chain.[0m
```


```output
{'messages': [HumanMessage(content="I'm Nemo!"),
  AIMessage(content='Hello Nemo! How can I assist you today?'),
  HumanMessage(content='What is my name?')],
 'output': 'Your name is Nemo!'}
```


If preferred, you can also wrap the agent executor in a [`RunnableWithMessageHistory`](/docs/how_to/message_history/) class to internally manage history messages. Let's redeclare it this way:


```python
agent = create_tool_calling_agent(chat, tools, prompt)

agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```

Then, because our agent executor has multiple outputs, we also have to set the `output_messages_key` property when initializing the wrapper:


```python
from langchain_community.chat_message_histories import ChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory

demo_ephemeral_chat_history_for_chain = ChatMessageHistory()

conversational_agent_executor = RunnableWithMessageHistory(
    agent_executor,
    lambda session_id: demo_ephemeral_chat_history_for_chain,
    input_messages_key="messages",
    output_messages_key="output",
)

conversational_agent_executor.invoke(
    {"messages": [HumanMessage("I'm Nemo!")]},
    {"configurable": {"session_id": "unused"}},
)
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3mHi Nemo! It's great to meet you. How can I assist you today?[0m

[1m> Finished chain.[0m
```


```output
{'messages': [HumanMessage(content="I'm Nemo!")],
 'output': "Hi Nemo! It's great to meet you. How can I assist you today?"}
```


And then if we rerun our wrapped agent executor:


```python
conversational_agent_executor.invoke(
    {"messages": [HumanMessage("What is my name?")]},
    {"configurable": {"session_id": "unused"}},
)
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3mYour name is Nemo! How can I assist you today, Nemo?[0m

[1m> Finished chain.[0m
```


```output
{'messages': [HumanMessage(content="I'm Nemo!"),
  AIMessage(content="Hi Nemo! It's great to meet you. How can I assist you today?"),
  HumanMessage(content='What is my name?')],
 'output': 'Your name is Nemo! How can I assist you today, Nemo?'}
```


This [LangSmith trace](https://smith.langchain.com/public/1a9f712a-7918-4661-b3ff-d979bcc2af42/r) shows what's going on under the hood.

## Further reading

Other types agents can also support conversational responses too - for more, check out the [agents section](/docs/tutorials/agents).

For more on tool usage, you can also check out [this use case section](/docs/how_to#tools).
