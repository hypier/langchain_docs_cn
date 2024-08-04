---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/exa_search.ipynb
---
# Exa Search

Exa (formerly Metaphor Search) is a search engine fully designed for use by LLMs. Search for documents on the internet using **natural language queries**, then retrieve **cleaned HTML content** from desired documents.

Unlike keyword-based search (Google), Exa's neural search capabilities allow it to semantically understand queries and return relevant documents. For example, we could search `"fascinating article about cats"` and compare the search results from [Google](https://www.google.com/search?q=fascinating+article+about+cats) and [Exa](https://search.exa.ai/search?q=fascinating%20article%20about%20cats&autopromptString=Here%20is%20a%20fascinating%20article%20about%20cats%3A). Google gives us SEO-optimized listicles based on the keyword "fascinating". Exa just works.

This notebook goes over how to use Exa Search with LangChain.

First, get an Exa API key and add it as an environment variable. Get 1000 free searches/month by [signing up here](https://dashboard.exa.ai/).


```python
import os

os.environ["EXA_API_KEY"] = "..."
```

And install the integration package


```python
%pip install --upgrade --quiet langchain-exa 

# and some deps for this notebook
%pip install --upgrade --quiet langchain langchain-openai langchain-community
```

## Using ExaSearchRetriever

ExaSearchRetriever is a retriever that uses Exa Search to retrieve relevant documents.

:::note

The `max_characters` parameter for **TextContentsOptions** used to be called `max_length` which is now deprecated. Make sure to use `max_characters` instead.

:::


```python
from langchain_core.prompts import PromptTemplate
from langchain_core.runnables import RunnableParallel, RunnablePassthrough
from langchain_exa import ExaSearchRetriever, TextContentsOptions
from langchain_openai import ChatOpenAI

# retrieve 5 documents, with content truncated at 1000 characters
retriever = ExaSearchRetriever(
    k=5, text_contents_options=TextContentsOptions(max_characters=200)
)

prompt = PromptTemplate.from_template(
    """Answer the following query based on the following context:
query: {query}
<context>
{context}
</context"""
)

llm = ChatOpenAI()

chain = (
    RunnableParallel({"context": retriever, "query": RunnablePassthrough()})
    | prompt
    | llm
)

chain.invoke("When is the best time to visit japan?")
```
```output
[Result(title='Find Us:', url='https://travelila.com/best-time-to-visit-japan/', id='UFLQGtanQffaDErhngnzgA', score=0.1865834891796112, published_date='2021-01-05', author=None, text='If you are planning to spend your next vacation in Japan, then hold your excitement a bit. It would help if you planned which places you will visit in Japan and the country’s best things. It’s entirel', highlights=None, highlight_scores=None), Result(title='When Is The Best Time of Year To Visit Japan?', url='https://boutiquejapan.com/when-is-the-best-time-of-year-to-visit-japan/', id='70b0IMuaQpshjpBpnwsfUg', score=0.17796635627746582, published_date='2022-09-26', author='Andres Zuleta', text='The good news for travelers is that there is no single best time of year to travel to Japan — yet this makes it hard to decide when to visit, as each season has its own special highlights.When plannin', highlights=None, highlight_scores=None), Result(title='Here is the Best Time to Visit Japan - Cooking Sun', url='https://www.cooking-sun.com/best-time-to-visit-japan/', id='2mh-xvoqGPT-ZRvX9GezNQ', score=0.17497511208057404, published_date='2018-12-17', author='Cooking Sun', text='Japan is a diverse and beautiful country that’s brimming with culture. For some travelers, visiting Japan is a dream come true, since it grazes bucket lists across the globe. One of the best parts abo', highlights=None, highlight_scores=None), Result(title='When to Visit Japan? Bests Times and 2023 Travel Tips', url='https://www.jrailpass.com/blog/when-visit-japan-times', id='KqCnY8fF-nc76n1wNpIo1Q', score=0.17359933257102966, published_date='2020-02-18', author='JRailPass', text='When is the best time to visit Japan? This is a question without a simple answer. Japan is a year-round destination, with interesting activities, attractions, and festivities throughout the year.Your ', highlights=None, highlight_scores=None), Result(title='Complete Guide To Visiting Japan In February 2023: Weather, What To See & Do | LIVE JAPAN travel guide', url='https://livejapan.com/en/article-a0002948/', id='i3nmekOdM8_VBxPfcJmxng', score=0.17215865850448608, published_date='2019-11-13', author='Lucio Maurizi', text='\n \n \n HOME\n Complete Guide To Visiting Japan In February 2023: Weather, What To See & Do\n \n \n \n \n \n \n Date published: 13 November 2019 \n Last updated: 26 January 2021 \n \n \n So you’re planning your tra', highlights=None, highlight_scores=None)]
```


```output
AIMessage(content='Based on the given context, there is no specific best time mentioned to visit Japan. Each season has its own special highlights, and Japan is a year-round destination with interesting activities, attractions, and festivities throughout the year. Therefore, the best time to visit Japan depends on personal preferences and the specific activities or events one wants to experience.')
```


## Using the Exa SDK as LangChain Agent Tools

The [Exa SDK](https://docs.exa.ai/) creates a client that can use the Exa API to perform three functions:
- `search`: Given a natural language search query, retrieve a list of search results.
- `find_similar`: Given a URL, retrieve a list of search results corresponding to webpages which are similar to the document at the provided URL.
- `get_content`: Given a list of document ids fetched from `search` or `find_similar`, get cleaned HTML content for each document.

We can use the `@tool` decorator and docstrings to create LangChain Tool wrappers that tell an LLM agent how to use Exa.


```python
%pip install --upgrade --quiet  langchain-exa
```


```python
from exa_py import Exa
from langchain_core.tools import tool

exa = Exa(api_key=os.environ["EXA_API_KEY"])


@tool
def search(query: str):
    """Search for a webpage based on the query."""
    return exa.search(f"{query}", use_autoprompt=True, num_results=5)


@tool
def find_similar(url: str):
    """Search for webpages similar to a given URL.
    The url passed in should be a URL returned from `search`.
    """
    return exa.find_similar(url, num_results=5)


@tool
def get_contents(ids: list[str]):
    """Get the contents of a webpage.
    The ids passed in should be a list of ids returned from `search`.
    """
    return exa.get_contents(ids)


tools = [search, get_contents, find_similar]
```

### Providing Exa Tools to an Agent

We can provide the Exa tools we just created to a LangChain `OpenAIFunctionsAgent`. When asked to `Summarize for me a fascinating article about cats`, the agent uses the `search` tool to perform a Exa search with an appropriate search query, uses the `get_contents` tool to perform Exa content retrieval, and then returns a summary of the retrieved content.


```python
from langchain.agents import AgentExecutor, OpenAIFunctionsAgent
from langchain_core.messages import SystemMessage
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(temperature=0)

system_message = SystemMessage(
    content="You are a web researcher who answers user questions by looking up information on the internet and retrieving contents of helpful documents. Cite your sources."
)

agent_prompt = OpenAIFunctionsAgent.create_prompt(system_message)
agent = OpenAIFunctionsAgent(llm=llm, tools=tools, prompt=agent_prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```


```python
agent_executor.run("Summarize for me a fascinating article about cats.")
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m
Invoking: `search` with `{'query': 'fascinating article about cats'}`


[0m[36;1m[1;3mTitle: How Much Would You Pay to Save Your Cat’s Life?
URL: https://www.theatlantic.com/magazine/archive/2022/12/paying-for-pet-critical-care-cost-health-insurance/671896/
ID: 8uIICapOJD68i8iKEaRfPg
Score: 0.17565323412418365
Published Date: 2022-11-20
Author: Sarah Zhang
Extract: None

Title: How Your Cat Is Making You Crazy
URL: https://www.theatlantic.com/magazine/archive/2012/03/how-your-cat-is-making-you-crazy/308873/
ID: tqgaWIm4JkU9wfINfE-52Q
Score: 0.1744839996099472
Published Date: 2012-02-06
Author: Kathleen McAuliffe
Extract: None

Title: The Feline Mystique
URL: https://www.mcsweeneys.net/articles/the-feline-mystique
ID: wq8uMC5BEjSmvA-NBSCTvQ
Score: 0.17415249347686768
Published Date: 2022-07-19
Author: Kathryn Baecht
Extract: None

Title: How Much Would You Pay to Save Your Cat’s Life?
URL: https://www.theatlantic.com/magazine/archive/2022/12/paying-for-pet-critical-care-cost-health-insurance/671896/?utm%5C_source=pocket-newtab-global-en-GB
ID: wpT7Ee9vkcJyDUHgOelD_w
Score: 0.17202475666999817
Published Date: 2022-11-20
Author: Sarah Zhang
Extract: None

Title: How Much Would You Pay to Save Your Cat’s Life?
URL: https://www.theatlantic.com/magazine/archive/2022/12/paying-for-pet-critical-care-cost-health-insurance/671896/?src=longreads
ID: IkNF5fN0F8B0W_-Boh567A
Score: 0.16996535658836365
Published Date: 2022-11-20
Author: Sarah Zhang
Extract: None

Autoprompt String: Here is a fascinating article about cats:[0m[32;1m[1;3m
Invoking: `get_contents` with `{'ids': ['8uIICapOJD68i8iKEaRfPg']}`


[0m[33;1m[1;3mID: 8uIICapOJD68i8iKEaRfPg
URL: https://www.theatlantic.com/magazine/archive/2022/12/paying-for-pet-critical-care-cost-health-insurance/671896/
Title: How Much Would You Pay to Save Your Cat’s Life?
Extract: <div><article><header><div><div><p>For $15,000, you can get your pet a new kidney.</p><div><address>By <a href="https://www.theatlantic.com/author/sarah-zhang/">Sarah Zhang</a><p>Photographs by Caroline Tompkins for <i>The Atlantic</i></p></address></div></div><div><figure><div></div><figcaption>Sherlock donated a kidney in 2019. (Caroline Tompkins for The Atlantic)</figcaption></figure></div></div></header><section><figure></figure><p><small><i>This article was featured in One Story to Read Today, a newsletter in which our editors recommend a single must-read from </i>The Atlantic<i>, Monday through Friday. </i><a href="https://www.theatlantic.com/newsletters/sign-up/one-story-to-read-today/"><i>Sign up for it here.</i></a><i>      </i></small></p><p>When I first met Strawberry, age 16, she was lying on her back, paws akimbo. Her cat belly was shaved bare, and black stitches ran several inches down her naked pink skin.</p><p>A radiologist squirted ultrasound goop on her abdomen while two veterinary students in dark-blue scrubs gently held down her legs—not that this was really necessary. Strawberry was too tired, too drugged, or simply too out of it from her surgery the previous day to protest. In the dim light of the radiology room, her pupils were dilated into deep black pools. She slowly turned her head toward me. She turned away. She looked around at the small crowd of doctors and students surrounding her, as if to wonder what on God’s green earth had happened for her to end up like this.</p><section><figure><a href="https://www.theatlantic.com/magazine/toc/2022/12/"></a></figure><div><h2>Explore the December 2022 Issue</h2><p>Check out more from this issue and find your next story to read.</p></div><a href="https://www.theatlantic.com/magazine/toc/2022/12/">View More</a></section><p>What had happened was that Strawberry had received a kidney transplant. A surgical team at the University of Georgia had shaved off patches of her long ginger fur, inserting catheters in her leg and neck to deliver the cocktail of drugs she would need during her hospital stay: anesthesia, painkillers, antibiotics, blood thinners, and immunosuppressants. Then a surgeon named Chad Schmiedt carefully cut down the midline of her belly—past the two shriveled kidneys that were no longer doing their job and almost to her groin. Next, he stitched into place a healthy new kidney, freshly retrieved from a living donor just hours earlier.</p><p>Schmiedt is one of only a few surgeons who perform transplants on cats, and is therefore one of the world’s foremost experts at connecting cat kidneys. When he first greeted me with a broad smile and a handshake, I was struck by how his large, callused hand engulfed mine. In the operating room, though, his hands work with microscopic precision, stitching up arteries and veins only millimeters wide. This is the hardest part, he told me, like sewing “wet rice paper.” Once the donor kidney was in place, it flushed pink and Schmiedt closed Strawberry back up. (As in human transplants, the old kidneys can stay in place.) It was then a matter of waiting for her to wake up and pee. She had done both by the time of her ultrasound.</p><p>Not that Strawberry could understand any of this—or that any cat understands why we humans insist on bringing them to vet offices to be poked and prodded by strangers. But without the transplant, she would die of kidney failure, an affliction akin to being gradually poisoned from within. Other treatments could slow her kidney disease, which is common in older cats, but they could not stop it. This is why Strawberry’s owner decided to spend $15,000 on a kidney—a last resort to save her life, or at least extend it.</p><p>I didn’t meet her owner in the hospital that day. Strawberry would need to be hospitalized for at least a week afterAuthor: None[0m[32;1m[1;3mHere is a fascinating article about cats titled "How Much Would You Pay to Save Your Cat’s Life?" by Sarah Zhang. The article discusses the cost of critical care for pets and the lengths that some owners are willing to go to save their cat's life. It highlights a case of a cat named Strawberry who underwent a kidney transplant, a procedure performed by one of the few surgeons who specialize in cat transplants. The article explores the emotional and financial decisions involved in providing life-saving treatments for pets. You can read the full article [here](https://www.theatlantic.com/magazine/archive/2022/12/paying-for-pet-critical-care-cost-health-insurance/671896/).[0m

[1m> Finished chain.[0m
```


```output
'Here is a fascinating article about cats titled "How Much Would You Pay to Save Your Cat’s Life?" by Sarah Zhang. The article discusses the cost of critical care for pets and the lengths that some owners are willing to go to save their cat\'s life. It highlights a case of a cat named Strawberry who underwent a kidney transplant, a procedure performed by one of the few surgeons who specialize in cat transplants. The article explores the emotional and financial decisions involved in providing life-saving treatments for pets. You can read the full article [here](https://www.theatlantic.com/magazine/archive/2022/12/paying-for-pet-critical-care-cost-health-insurance/671896/).'
```


## Advanced Exa Features

Exa supports powerful filters by domain and date. We can provide a more powerful `search` tool to the agent that lets it decide to apply filters if they are useful for the objective. See all of Exa's search features [here](https://github.com/metaphorsystems/metaphor-python/).

[//]: # "TODO(erick): switch metaphor github link to exa github link when sdk published"


```python
from exa_py import Exa
from langchain_core.tools import tool

exa = Exa(api_key=os.environ["Exa_API_KEY"])


@tool
def search(query: str, include_domains=None, start_published_date=None):
    """Search for a webpage based on the query.
    Set the optional include_domains (list[str]) parameter to restrict the search to a list of domains.
    Set the optional start_published_date (str) parameter to restrict the search to documents published after the date (YYYY-MM-DD).
    """
    return exa.search_and_contents(
        f"{query}",
        use_autoprompt=True,
        num_results=5,
        include_domains=include_domains,
        start_published_date=start_published_date,
    )


@tool
def find_similar(url: str):
    """Search for webpages similar to a given URL.
    The url passed in should be a URL returned from `search`.
    """
    return exa.find_similar_and_contents(url, num_results=5)


@tool
def get_contents(ids: list[str]):
    """Get the contents of a webpage.
    The ids passed in should be a list of ids returned from `search`.
    """
    return exa.get_contents(ids)


tools = [search, get_contents, find_similar]
```

Now we ask the agent to summarize an article with constraints on domain and publish date. We will use a GPT-4 agent for extra powerful reasoning capability to support more complex tool usage.

The agent correctly uses the search filters to find an article with the desired constraints, and once again retrieves the content and returns a summary.


```python
from langchain.agents import AgentExecutor, OpenAIFunctionsAgent
from langchain_core.messages import SystemMessage
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(temperature=0, model="gpt-4")

system_message = SystemMessage(
    content="You are a web researcher who answers user questions by looking up information on the internet and retrieving contents of helpful documents. Cite your sources."
)

agent_prompt = OpenAIFunctionsAgent.create_prompt(system_message)
agent = OpenAIFunctionsAgent(llm=llm, tools=tools, prompt=agent_prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```


```python
agent_executor.run(
    "Summarize for me an interesting article about AI from lesswrong.com published after October 2023."
)
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m
Invoking: `search` with `{'query': 'AI article', 'include_domains': ['lesswrong.com'], 'start_published_date': '2023-10-01'}`


[0m[36;1m[1;3mTitle: A dialectical view of the history of AI, Part 1: We’re only in the antithesis phase. [A synthesis is in the future.]
URL: https://www.lesswrong.com/posts/K9GP2ZSsugRAGfFns/a-dialectical-view-of-the-history-of-ai-part-1-we-re-only-in
ID: 45-Cs6bdV4_HB8epTO0utg
Score: 0.14216266572475433
Published Date: 2023-11-16
Author: Bill Benzon
Extract: None

Title: Possible OpenAI's Q* breakthrough and DeepMind's AlphaGo-type systems plus LLMs
URL: https://www.lesswrong.com/posts/JnM3EHegiBePeKkLc/possible-openai-s-q-breakthrough-and-deepmind-s-alphago-type
ID: BPJkDFi_bcpo7T19zlj0Ag
Score: 0.13007280230522156
Published Date: 2023-11-23
Author: Burny
Extract: None

Title: Architects of Our Own Demise: We Should Stop Developing AI
URL: https://www.lesswrong.com/posts/bHHrdXwrCj2LRa2sW/architects-of-our-own-demise-we-should-stop-developing-ai
ID: IMLGjczHCLOuoB6w_vemkA
Score: 0.12803198397159576
Published Date: 2023-10-26
Author: Roko
Extract: None

Title: "Benevolent [ie, Ruler] AI is a bad idea" and a suggested alternative
URL: https://www.lesswrong.com/posts/bwL63TrCo4RaHc3WJ/benevolent-ie-ruler-ai-is-a-bad-idea-and-a-suggested
ID: RvRnTH2cpV186d98SFbb9w
Score: 0.12487411499023438
Published Date: 2023-11-19
Author: The Gears
Extract: None

Title: "Benevolent [Ruler] AI is a bad idea" and a suggested alternative
URL: https://www.lesswrong.com/posts/bwL63TrCo4RaHc3WJ/benevolent-ruler-ai-is-a-bad-idea-and-a-suggested
ID: PjwqS66am-duoo_2ueRpQw
Score: 0.12358281761407852
Published Date: 2023-11-19
Author: The Gears
Extract: None

Autoprompt String: Here is an interesting article about AI:[0m[32;1m[1;3m
Invoking: `get_contents` with `{'ids': ['45-Cs6bdV4_HB8epTO0utg']}`


[0m[33;1m[1;3mID: 45-Cs6bdV4_HB8epTO0utg
URL: https://www.lesswrong.com/posts/K9GP2ZSsugRAGfFns/a-dialectical-view-of-the-history-of-ai-part-1-we-re-only-in
Title: A dialectical view of the history of AI, Part 1: We’re only in the antithesis phase. [A synthesis is in the future.]
Extract: <div><div><p><i>Cross-posted </i><a href="https://new-savanna.blogspot.com/2023/11/a-dialectical-view-of-history-of-ai.html"><i>from New Savanna</i></a><i>.</i></p><p>The idea that <a href="https://www.britannica.com/topic/philosophy-of-history/History-as-a-process-of-dialectical-change-Hegel-and-Marx">history proceeds by way of dialectical change</a> is due primarily to Hegel and Marx. While I read bit of both early in my career, I haven’t been deeply influenced by either of them. Nonetheless I find the notion of dialectical change working out through history to be a useful way of thinking about the history of AI. Because it implies that that history is more than just one thing of another.</p><p>This dialectical process is generally schematized as a movement from a thesis, to an antithesis, and finally, to a synthesis on a “higher level,” whatever that is. The technical term is <i>Aufhebung</i>. Wikipedia:</p><blockquote><p>In Hegel, the term <i>Aufhebung</i> has the apparently contradictory implications of both preserving and changing, and eventually advancement (the German verb <i>aufheben</i> means "to cancel", "to keep" and "to pick up"). The tension between these senses suits what Hegel is trying to talk about. In sublation, a term or concept is both preserved and changed through its dialectical interplay with another term or concept. Sublation is the motor by which the dialectic functions.</p></blockquote><p>So, why do I think the history of AI is best conceived in this way? The first era, THESIS, running from the 1950s up through and into the 1980s, was based on top-down deductive <i>symbolic</i> methods. The second era, ANTITHESIS, which began its ascent in the 1990s and now reigns, is based on bottom-up <i>statistical</i> methods. These are conceptually and computationally quite different, opposite, if you will. As for the third era, SYNTHESIS, well, we don’t even know if there will be a third era. Perhaps the second, the current, era will take us <i>all the way</i>, whatever that means. Color me skeptical. I believe there will be a third era, and that it will involve a synthesis of conceptual ideas computational techniques from the previous eras.</p><p>Note, though, that I will be concentrating on efforts to model language. In the first place, that’s what I know best. More importantly, however, it is the work on language that is currently evoking the most fevered speculations about the future of AI.</p><p>Let’s take a look. Find a comfortable chair, adjust the lighting, pour yourself a drink, sit back, relax, and read. This is going to take a while.</p><p><strong>Symbolic AI: Thesis</strong></p><p>The pursuit of artificial intelligence started back in the 1950s it began with certain ideas and certain computational capabilities. The latter were crude and radically underpowered by today’s standards. As for the ideas, we need two more or less independent starting points. One gives us the term “artificial intelligence” (AI), which John McCarthy coined in connection with a conference held at Dartmouth in 1956. The other is associated with the pursuit of machine translation (MT) which, in the United States, meant translating Russian technical documents into English. MT was funded primarily by the Defense Department.</p><p>The goal of MT was practical, relentlessly practical. There was no talk of intelligence and Turing tests and the like. The only thing that mattered was being able to take a Russian text, feed it into a computer, and get out a competent English translation of that text. Promises was made, but little was delivered. The Defense Department pulled the plug on that work in the mid-1960s. Researchers in MT then proceeded to rebrand themselves as investigators ofAuthor: None[0m[32;1m[1;3mThe article titled "A dialectical view of the history of AI, Part 1: We’re only in the antithesis phase. [A synthesis is in the future.]" on LessWrong provides a unique perspective on the evolution of AI. The author, Bill Benzon, uses the concept of dialectical change, primarily attributed to Hegel and Marx, to explain the history of AI. 

The dialectical process is described as a movement from a thesis, to an antithesis, and finally, to a synthesis on a higher level. The author applies this concept to AI by identifying three eras. The first era, the THESIS, ran from the 1950s to the 1980s and was based on top-down deductive symbolic methods. The second era, the ANTITHESIS, began in the 1990s and is based on bottom-up statistical methods. These two eras are conceptually and computationally quite different. 

As for the third era, the SYNTHESIS, the author speculates that it will involve a synthesis of conceptual ideas and computational techniques from the previous eras. However, it is unclear if this era will occur or if the current era will take us "all the way". 

The author also notes that the focus of the article is on efforts to model language, as it is the work on language that is currently evoking the most speculation about the future of AI. The article then delves into the history of symbolic AI, starting from the 1950s when the term "artificial intelligence" was coined by John McCarthy. 

The article provides a comprehensive and thought-provoking view on the history and potential future of AI, particularly in the context of language modeling. 

Source: [LessWrong](https://www.lesswrong.com/posts/K9GP2ZSsugRAGfFns/a-dialectical-view-of-the-history-of-ai-part-1-we-re-only-in)[0m

[1m> Finished chain.[0m
```


```output
'The article titled "A dialectical view of the history of AI, Part 1: We’re only in the antithesis phase. [A synthesis is in the future.]" on LessWrong provides a unique perspective on the evolution of AI. The author, Bill Benzon, uses the concept of dialectical change, primarily attributed to Hegel and Marx, to explain the history of AI. \n\nThe dialectical process is described as a movement from a thesis, to an antithesis, and finally, to a synthesis on a higher level. The author applies this concept to AI by identifying three eras. The first era, the THESIS, ran from the 1950s to the 1980s and was based on top-down deductive symbolic methods. The second era, the ANTITHESIS, began in the 1990s and is based on bottom-up statistical methods. These two eras are conceptually and computationally quite different. \n\nAs for the third era, the SYNTHESIS, the author speculates that it will involve a synthesis of conceptual ideas and computational techniques from the previous eras. However, it is unclear if this era will occur or if the current era will take us "all the way". \n\nThe author also notes that the focus of the article is on efforts to model language, as it is the work on language that is currently evoking the most speculation about the future of AI. The article then delves into the history of symbolic AI, starting from the 1950s when the term "artificial intelligence" was coined by John McCarthy. \n\nThe article provides a comprehensive and thought-provoking view on the history and potential future of AI, particularly in the context of language modeling. \n\nSource: [LessWrong](https://www.lesswrong.com/posts/K9GP2ZSsugRAGfFns/a-dialectical-view-of-the-history-of-ai-part-1-we-re-only-in)'
```



## Related

- Tool [conceptual guide](/docs/concepts/#tools)
- Tool [how-to guides](/docs/how_to/#tools)
