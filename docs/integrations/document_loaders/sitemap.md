---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/sitemap.ipynb
---
# Sitemap

Extends from the `WebBaseLoader`, `SitemapLoader` loads a sitemap from a given URL, and then scrape and load all pages in the sitemap, returning each page as a Document.

The scraping is done concurrently.  There are reasonable limits to concurrent requests, defaulting to 2 per second.  If you aren't concerned about being a good citizen, or you control the scrapped server, or don't care about load. Note, while this will speed up the scraping process, but it may cause the server to block you.  Be careful!


```python
%pip install --upgrade --quiet  nest_asyncio
```


```python
# fixes a bug with asyncio and jupyter
import nest_asyncio

nest_asyncio.apply()
```


```python
from langchain_community.document_loaders.sitemap import SitemapLoader
```


```python
sitemap_loader = SitemapLoader(web_path="https://api.python.langchain.com/sitemap.xml")

docs = sitemap_loader.load()
```

You can change the `requests_per_second` parameter to increase the max concurrent requests. and use `requests_kwargs` to pass kwargs when send requests.


```python
sitemap_loader.requests_per_second = 2
# Optional: avoid `[SSL: CERTIFICATE_VERIFY_FAILED]` issue
sitemap_loader.requests_kwargs = {"verify": False}
```


```python
docs[0]
```



```output
Document(page_content='\n\n\n\n\n\n\n\n\n\nLangChain Python API Reference Documentation.\n\n\nYou will be automatically redirected to the new location of this page.\n\n', metadata={'source': 'https://api.python.langchain.com/en/stable/', 'loc': 'https://api.python.langchain.com/en/stable/', 'lastmod': '2024-02-09T01:10:49.422114+00:00', 'changefreq': 'weekly', 'priority': '1'})
```


## Filtering sitemap URLs

Sitemaps can be massive files, with thousands of URLs.  Often you don't need every single one of them.  You can filter the URLs by passing a list of strings or regex patterns to the `filter_urls` parameter.  Only URLs that match one of the patterns will be loaded.


```python
loader = SitemapLoader(
    web_path="https://api.python.langchain.com/sitemap.xml",
    filter_urls=["https://api.python.langchain.com/en/latest"],
)
documents = loader.load()
```


```python
documents[0]
```



```output
Document(page_content='\n\n\n\n\n\n\n\n\n\nLangChain Python API Reference Documentation.\n\n\nYou will be automatically redirected to the new location of this page.\n\n', metadata={'source': 'https://api.python.langchain.com/en/latest/', 'loc': 'https://api.python.langchain.com/en/latest/', 'lastmod': '2024-02-12T05:26:10.971077+00:00', 'changefreq': 'daily', 'priority': '0.9'})
```


## Add custom scraping rules

The `SitemapLoader` uses `beautifulsoup4` for the scraping process, and it scrapes every element on the page by default. The `SitemapLoader` constructor accepts a custom scraping function. This feature can be helpful to tailor the scraping process to your specific needs; for example, you might want to avoid scraping headers or navigation elements.

 The following example shows how to develop and use a custom function to avoid navigation and header elements.

Import the `beautifulsoup4` library and define the custom function.


```python
pip install beautifulsoup4
```


```python
from bs4 import BeautifulSoup


def remove_nav_and_header_elements(content: BeautifulSoup) -> str:
    # Find all 'nav' and 'header' elements in the BeautifulSoup object
    nav_elements = content.find_all("nav")
    header_elements = content.find_all("header")

    # Remove each 'nav' and 'header' element from the BeautifulSoup object
    for element in nav_elements + header_elements:
        element.decompose()

    return str(content.get_text())
```

Add your custom function to the `SitemapLoader` object.


```python
loader = SitemapLoader(
    "https://api.python.langchain.com/sitemap.xml",
    filter_urls=["https://api.python.langchain.com/en/latest/"],
    parsing_function=remove_nav_and_header_elements,
)
```

## Local Sitemap

The sitemap loader can also be used to load local files.


```python
sitemap_loader = SitemapLoader(web_path="example_data/sitemap.xml", is_local=True)

docs = sitemap_loader.load()
```


## Related

- Document loader [conceptual guide](/docs/concepts/#document-loaders)
- Document loader [how-to guides](/docs/how_to/#document-loaders)
