---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/sql_database.ipynb
---
# SQL Database

:::note
The `SQLDatabase` adapter utility is a wrapper around a database connection.

For talking to SQL databases, it uses the [SQLAlchemy] Core API .
:::


This notebook shows how to use the utility to access an SQLite database.
It uses the example [Chinook Database], and demonstrates those features:

- Query using SQL
- Query using SQLAlchemy selectable
- Fetch modes `cursor`, `all`, and `one`
- Bind query parameters

[Chinook Database]: https://github.com/lerocha/chinook-database
[SQLAlchemy]: https://www.sqlalchemy.org/


You can use the `Tool` or `@tool` decorator to create a tool from this utility.


::: {.callout-caution}
If creating a tool from the SQLDatbase utility and combining it with an LLM or exposing it to an end user
remember to follow good security practices.

See security information: https://python.langchain.com/docs/security
:::


```python
!wget 'https://github.com/lerocha/chinook-database/releases/download/v1.4.2/Chinook_Sqlite.sql'
```


```python
!sqlite3 -bail -cmd '.read Chinook_Sqlite.sql' -cmd 'SELECT * FROM Artist LIMIT 12;' -cmd '.quit'
```
```output
1|AC/DC
2|Accept
3|Aerosmith
4|Alanis Morissette
5|Alice In Chains
6|Antônio Carlos Jobim
7|Apocalyptica
8|Audioslave
9|BackBeat
10|Billy Cobham
11|Black Label Society
12|Black Sabbath
```

```python
!sqlite3 -bail -cmd '.read Chinook_Sqlite.sql' -cmd '.save Chinook.db' -cmd '.quit'
```

## Initialize Database


```python
from pprint import pprint

import sqlalchemy as sa
from langchain_community.utilities import SQLDatabase

db = SQLDatabase.from_uri("sqlite:///Chinook.db")
```

## Query as cursor

The fetch mode `cursor` returns results as SQLAlchemy's
`CursorResult` instance.


```python
result = db.run("SELECT * FROM Artist LIMIT 12;", fetch="cursor")
print(type(result))
pprint(list(result.mappings()))
```
```output
<class 'sqlalchemy.engine.cursor.CursorResult'>
[{'ArtistId': 1, 'Name': 'AC/DC'},
 {'ArtistId': 2, 'Name': 'Accept'},
 {'ArtistId': 3, 'Name': 'Aerosmith'},
 {'ArtistId': 4, 'Name': 'Alanis Morissette'},
 {'ArtistId': 5, 'Name': 'Alice In Chains'},
 {'ArtistId': 6, 'Name': 'Antônio Carlos Jobim'},
 {'ArtistId': 7, 'Name': 'Apocalyptica'},
 {'ArtistId': 8, 'Name': 'Audioslave'},
 {'ArtistId': 9, 'Name': 'BackBeat'},
 {'ArtistId': 10, 'Name': 'Billy Cobham'},
 {'ArtistId': 11, 'Name': 'Black Label Society'},
 {'ArtistId': 12, 'Name': 'Black Sabbath'}]
```
## Query as string payload

The fetch modes `all` and `one` return results in string format.


```python
result = db.run("SELECT * FROM Artist LIMIT 12;", fetch="all")
print(type(result))
print(result)
```
```output
<class 'str'>
[(1, 'AC/DC'), (2, 'Accept'), (3, 'Aerosmith'), (4, 'Alanis Morissette'), (5, 'Alice In Chains'), (6, 'Antônio Carlos Jobim'), (7, 'Apocalyptica'), (8, 'Audioslave'), (9, 'BackBeat'), (10, 'Billy Cobham'), (11, 'Black Label Society'), (12, 'Black Sabbath')]
```

```python
result = db.run("SELECT * FROM Artist LIMIT 12;", fetch="one")
print(type(result))
print(result)
```
```output
<class 'str'>
[(1, 'AC/DC')]
```
## Query with parameters

In order to bind query parameters, use the optional `parameters` argument.


```python
result = db.run(
    "SELECT * FROM Artist WHERE Name LIKE :search;",
    parameters={"search": "p%"},
    fetch="cursor",
)
pprint(list(result.mappings()))
```
```output
[{'ArtistId': 35, 'Name': 'Pedro Luís & A Parede'},
 {'ArtistId': 115, 'Name': 'Page & Plant'},
 {'ArtistId': 116, 'Name': 'Passengers'},
 {'ArtistId': 117, 'Name': "Paul D'Ianno"},
 {'ArtistId': 118, 'Name': 'Pearl Jam'},
 {'ArtistId': 119, 'Name': 'Peter Tosh'},
 {'ArtistId': 120, 'Name': 'Pink Floyd'},
 {'ArtistId': 121, 'Name': 'Planet Hemp'},
 {'ArtistId': 186, 'Name': 'Pedro Luís E A Parede'},
 {'ArtistId': 256, 'Name': 'Philharmonia Orchestra & Sir Neville Marriner'},
 {'ArtistId': 275, 'Name': 'Philip Glass Ensemble'}]
```
## Query with SQLAlchemy selectable

Other than plain-text SQL statements, the adapter also accepts SQLAlchemy selectables.


```python
# In order to build a selectable on SA's Core API, you need a table definition.
metadata = sa.MetaData()
artist = sa.Table(
    "Artist",
    metadata,
    sa.Column("ArtistId", sa.INTEGER, primary_key=True),
    sa.Column("Name", sa.TEXT),
)

# Build a selectable with the same semantics of the recent query.
query = sa.select(artist).where(artist.c.Name.like("p%"))
result = db.run(query, fetch="cursor")
pprint(list(result.mappings()))
```
```output
[{'ArtistId': 35, 'Name': 'Pedro Luís & A Parede'},
 {'ArtistId': 115, 'Name': 'Page & Plant'},
 {'ArtistId': 116, 'Name': 'Passengers'},
 {'ArtistId': 117, 'Name': "Paul D'Ianno"},
 {'ArtistId': 118, 'Name': 'Pearl Jam'},
 {'ArtistId': 119, 'Name': 'Peter Tosh'},
 {'ArtistId': 120, 'Name': 'Pink Floyd'},
 {'ArtistId': 121, 'Name': 'Planet Hemp'},
 {'ArtistId': 186, 'Name': 'Pedro Luís E A Parede'},
 {'ArtistId': 256, 'Name': 'Philharmonia Orchestra & Sir Neville Marriner'},
 {'ArtistId': 275, 'Name': 'Philip Glass Ensemble'}]
```
## Query with execution options

It is possible to augment the statement invocation with custom execution options.
For example, when applying a schema name translation, subsequent statements will
fail, because they try to hit a non-existing table.


```python
query = sa.select(artist).where(artist.c.Name.like("p%"))
db.run(query, fetch="cursor", execution_options={"schema_translate_map": {None: "bar"}})
```


## Related

- Tool [conceptual guide](/docs/concepts/#tools)
- Tool [how-to guides](/docs/how_to/#tools)
