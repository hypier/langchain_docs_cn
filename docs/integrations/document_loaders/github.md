---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/github.ipynb
---
# GitHub

This notebooks shows how you can load issues and pull requests (PRs) for a given repository on [GitHub](https://github.com/). Also shows how you can load github files for a given repository on [GitHub](https://github.com/). We will use the LangChain Python repository as an example.

## Setup access token

To access the GitHub API, you need a personal access token - you can set up yours here: https://github.com/settings/tokens?type=beta. You can either set this token as the environment variable ``GITHUB_PERSONAL_ACCESS_TOKEN`` and it will be automatically pulled in, or you can pass it in directly at initialization as the ``access_token`` named parameter.


```python
# If you haven't set your access token as an environment variable, pass it in here.
from getpass import getpass

ACCESS_TOKEN = getpass()
```

## Load Issues and PRs


```python
from langchain_community.document_loaders import GitHubIssuesLoader
```


```python
loader = GitHubIssuesLoader(
    repo="langchain-ai/langchain",
    access_token=ACCESS_TOKEN,  # delete/comment out this argument if you've set the access token as an env var.
    creator="UmerHA",
)
```

Let's load all issues and PRs created by "UmerHA".

Here's a list of all filters you can use:
- include_prs
- milestone
- state
- assignee
- creator
- mentioned
- labels
- sort
- direction
- since

For more info, see https://docs.github.com/en/rest/issues/issues?apiVersion=2022-11-28#list-repository-issues.


```python
docs = loader.load()
```


```python
print(docs[0].page_content)
print(docs[0].metadata)
```

## Only load issues

By default, the GitHub API returns considers pull requests to also be issues. To only get 'pure' issues (i.e., no pull requests), use `include_prs=False`


```python
loader = GitHubIssuesLoader(
    repo="langchain-ai/langchain",
    access_token=ACCESS_TOKEN,  # delete/comment out this argument if you've set the access token as an env var.
    creator="UmerHA",
    include_prs=False,
)
docs = loader.load()
```


```python
print(docs[0].page_content)
print(docs[0].metadata)
```

## Load Github File Content

For below code, loads all markdown file in rpeo `langchain-ai/langchain`


```python
from langchain.document_loaders import GithubFileLoader
```


```python
loader = GithubFileLoader(
    repo="langchain-ai/langchain",  # the repo name
    access_token=ACCESS_TOKEN,
    github_api_url="https://api.github.com",
    file_filter=lambda file_path: file_path.endswith(
        ".md"
    ),  # load all markdowns files.
)
documents = loader.load()
```

example output of one of document: 

```json
documents.metadata: 
    {
      "path": "README.md",
      "sha": "82f1c4ea88ecf8d2dfsfx06a700e84be4",
      "source": "https://github.com/langchain-ai/langchain/blob/master/README.md"
    }
documents.content:
    mock content
```


## Related

- Document loader [conceptual guide](/docs/concepts/#document-loaders)
- Document loader [how-to guides](/docs/how_to/#document-loaders)