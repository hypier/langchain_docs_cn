---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/cassandra_database.ipynb
---

# Cassandra数据库

>`Apache Cassandra®` 是一个广泛使用的数据库，用于存储事务应用数据。大型语言模型中功能和工具的引入为现有数据在生成式人工智能应用中的一些激动人心的用例打开了大门。

>`Cassandra数据库` 工具包使AI工程师能够高效地将代理与Cassandra数据集成，提供以下功能：
> - 通过优化查询实现快速数据访问。大多数查询应在个位数毫秒或更短时间内完成。
> - 模式自省以增强LLM推理能力
> - 兼容多种Cassandra部署，包括Apache Cassandra®、DataStax Enterprise™和DataStax Astra™
> - 目前，该工具包仅限于SELECT查询和模式自省操作。（安全第一）

有关创建Cassandra DB代理的更多信息，请参见 [CQL代理食谱](https://github.com/langchain-ai/langchain/blob/master/cookbook/cql_agent.ipynb)

## 快速开始
 - 安装 `cassio` 库
 - 为您要连接的 Cassandra 数据库设置环境变量
 - 初始化 `CassandraDatabase`
 - 使用 `toolkit.get_tools()` 将工具传递给您的代理
 - 放轻松，看看它为您完成所有工作

## 操作理论

`Cassandra Query Language (CQL)` 是与 Cassandra 数据库交互的主要 *以人为中心* 的方式。虽然在生成查询时提供了一定的灵活性，但它需要对 Cassandra 数据建模最佳实践的了解。LLM 函数调用使代理能够推理并选择工具以满足请求。使用 LLM 的代理在选择合适的工具包或工具链时应使用 Cassandra 特定的逻辑进行推理。这减少了在 LLM 被迫提供自上而下解决方案时引入的随机性。你希望 LLM 完全不受限制地访问你的数据库吗？是的。可能不行。为此，我们提供了一个提示，用于在为代理构建问题时使用：

你是一个 Apache Cassandra 专家查询分析机器人，具有以下功能和规则：
 - 你将接受最终用户关于在数据库中查找特定数据的问题。
 - 你将检查数据库的模式并创建查询路径。
 - 你将向用户提供正确的查询，以查找他们所需的数据，展示查询路径提供的步骤。
 - 你将使用最佳实践来查询 Apache Cassandra，使用分区键和聚簇列。
 - 避免在查询中使用 ALLOW FILTERING。
 - 目标是找到查询路径，因此可能需要查询其他表以获得最终答案。

以下是 JSON 格式的查询路径示例：

```json
 {
  "query_paths": [
    {
      "description": "直接查询用户表，使用电子邮件",
      "steps": [
        {
          "table": "user_credentials",
          "query": 
             "SELECT userid FROM user_credentials WHERE email = 'example@example.com';"
        },
        {
          "table": "users",
          "query": "SELECT * FROM users WHERE userid = ?;"
        }
      ]
    }
  ]
}
```

## 提供的工具

### `cassandra_db_schema`
收集连接数据库或特定模式的所有模式信息。当代理确定操作时，这一点至关重要。

### `cassandra_db_select_table_data`
从特定的键空间和表中选择数据。代理可以传递参数以用于谓词和返回记录数量的限制。

### `cassandra_db_query`
`cassandra_db_select_table_data` 的实验性替代方案，它接受由代理完全形成的查询字符串，而不是参数。 *警告*: 这可能导致不寻常的查询，这些查询的性能可能不佳（甚至无法工作）。此功能可能会在未来的版本中被移除。如果它有一些很酷的功能，我们也想知道。你永远不知道！

## 环境设置

安装以下 Python 模块：

```bash
pip install ipykernel python-dotenv cassio langchain_openai langchain langchain-community langchainhub
```

### .env 文件
连接通过 `cassio` 使用 `auto=True` 参数，笔记本使用 OpenAI。您应相应地创建一个 `.env` 文件。

对于 Cassandra，设置：
```bash
CASSANDRA_CONTACT_POINTS
CASSANDRA_USERNAME
CASSANDRA_PASSWORD
CASSANDRA_KEYSPACE
```

对于 Astra，设置：
```bash
ASTRA_DB_APPLICATION_TOKEN
ASTRA_DB_DATABASE_ID
ASTRA_DB_KEYSPACE
```

例如：

```bash
# 连接到 Astra：
ASTRA_DB_DATABASE_ID=a1b2c3d4-...
ASTRA_DB_APPLICATION_TOKEN=AstraCS:...
ASTRA_DB_KEYSPACE=notebooks

# 还需设置 
OPENAI_API_KEY=sk-....
```

（您也可以修改下面的代码以直接连接 `cassio`。）

```python
from dotenv import load_dotenv

load_dotenv(override=True)
```

```python
# 导入必要的库
import os

import cassio
from langchain import hub
from langchain.agents import AgentExecutor, create_openai_tools_agent
from langchain_community.agent_toolkits.cassandra_database.toolkit import (
    CassandraDatabaseToolkit,
)
from langchain_community.tools.cassandra_database.prompt import QUERY_PATH_PROMPT
from langchain_community.tools.cassandra_database.tool import (
    GetSchemaCassandraDatabaseTool,
    GetTableDataCassandraDatabaseTool,
    QueryCassandraDatabaseTool,
)
from langchain_community.utilities.cassandra_database import CassandraDatabase
from langchain_openai import ChatOpenAI
```

## 连接到 Cassandra 数据库


```python
cassio.init(auto=True)
session = cassio.config.resolve_session()
if not session:
    raise Exception(
        "检查环境配置或手动配置 cassio 连接参数"
    )
```


```python
# 测试数据准备

session = cassio.config.resolve_session()

session.execute("""DROP KEYSPACE IF EXISTS langchain_agent_test; """)

session.execute(
    """
CREATE KEYSPACE if not exists langchain_agent_test 
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};
"""
)

session.execute(
    """
    CREATE TABLE IF NOT EXISTS langchain_agent_test.user_credentials (
    user_email text PRIMARY KEY,
    user_id UUID,
    password TEXT
);
"""
)

session.execute(
    """
    CREATE TABLE IF NOT EXISTS langchain_agent_test.users (
    id UUID PRIMARY KEY,
    name TEXT,
    email TEXT
);"""
)

session.execute(
    """
    CREATE TABLE IF NOT EXISTS langchain_agent_test.user_videos ( 
    user_id UUID,
    video_id UUID,
    title TEXT,
    description TEXT,
    PRIMARY KEY (user_id, video_id)
);
"""
)

user_id = "522b1fe2-2e36-4cef-a667-cd4237d08b89"
video_id = "27066014-bad7-9f58-5a30-f63fe03718f6"

session.execute(
    f"""
    INSERT INTO langchain_agent_test.user_credentials (user_id, user_email) 
    VALUES ({user_id}, 'patrick@datastax.com');
"""
)

session.execute(
    f"""
    INSERT INTO langchain_agent_test.users (id, name, email) 
    VALUES ({user_id}, 'Patrick McFadin', 'patrick@datastax.com');
"""
)

session.execute(
    f"""
    INSERT INTO langchain_agent_test.user_videos (user_id, video_id, title)
    VALUES ({user_id}, {video_id}, '使用 Langflow 在 5 分钟内构建 LangChain LLM 应用程序');
"""
)

session.set_keyspace("langchain_agent_test")
```


```python
# 创建一个 CassandraDatabase 实例
# 使用 cassio 会话连接到数据库
db = CassandraDatabase()

# 创建 Cassandra 数据库工具
query_tool = QueryCassandraDatabaseTool(db=db)
schema_tool = GetSchemaCassandraDatabaseTool(db=db)
select_data_tool = GetTableDataCassandraDatabaseTool(db=db)
```


```python
# 选择将驱动代理的 LLM
# 仅某些模型支持此功能
llm = ChatOpenAI(temperature=0, model="gpt-4-1106-preview")
toolkit = CassandraDatabaseToolkit(db=db)

tools = toolkit.get_tools()

print("可用工具:")
for tool in tools:
    print(tool.name + "\t- " + tool.description)
```
```output
可用工具:
cassandra_db_schema	- 
    此工具的输入是一个 keyspace 名称，输出是 Apache Cassandra 表的表描述。
    如果查询不正确，将返回一条错误消息。
    如果返回错误，请告知用户该 keyspace 不存在并停止。

cassandra_db_query	- 
    对数据库执行 CQL 查询并获取结果。
    如果查询不正确，将返回一条错误消息。
    如果返回错误，请重写查询，检查查询并重试。

cassandra_db_select_table_data	- 
    从 Apache Cassandra 数据库中的表获取数据的工具。 
    使用 WHERE 子句指定使用主键的查询谓词。 空谓词将返回所有行。 尽量避免这种情况。 
    使用 limit 指定要返回的行数。 空 limit 将返回所有行。
```

```python
prompt = hub.pull("hwchase17/openai-tools-agent")

# 构建 OpenAI 工具代理
agent = create_openai_tools_agent(llm, tools, prompt)
```


```python
input = (
    QUERY_PATH_PROMPT
    + "\n\n这是您的任务：查找电子邮件地址为 'patrick@datastax.com' 的用户上传到 langchain_agent_test keyspace 的所有视频。"
)

agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

response = agent_executor.invoke({"input": input})

print(response["output"])
```
```output


[1m> 进入新的 AgentExecutor 链...[0m
[32;1m[1;3m
调用: `cassandra_db_schema`，参数为 `{'keyspace': 'langchain_agent_test'}`


[0m[36;1m[1;3m表名: user_credentials
- Keyspace: langchain_agent_test
- 列
  - password (text)
  - user_email (text)
  - user_id (uuid)
- 分区键: (user_email)
- 聚簇键: 

表名: user_videos
- Keyspace: langchain_agent_test
- 列
  - description (text)
  - title (text)
  - user_id (uuid)
  - video_id (uuid)
- 分区键: (user_id)
- 聚簇键: (video_id asc)


表名: users
- Keyspace: langchain_agent_test
- 列
  - email (text)
  - id (uuid)
  - name (text)
- 分区键: (id)
- 聚簇键: 

[0m[32;1m[1;3m
调用: `cassandra_db_select_table_data`，参数为 `{'keyspace': 'langchain_agent_test', 'table': 'user_credentials', 'predicate': "user_email = 'patrick@datastax.com'", 'limit': 1}`


[0m[38;5;200m[1;3mRow(user_email='patrick@datastax.com', password=None, user_id=UUID('522b1fe2-2e36-4cef-a667-cd4237d08b89'))[0m[32;1m[1;3m
调用: `cassandra_db_select_table_data`，参数为 `{'keyspace': 'langchain_agent_test', 'table': 'user_videos', 'predicate': 'user_id = 522b1fe2-2e36-4cef-a667-cd4237d08b89', 'limit': 10}`


[0m[38;5;200m[1;3mRow(user_id=UUID('522b1fe2-2e36-4cef-a667-cd4237d08b89'), video_id=UUID('27066014-bad7-9f58-5a30-f63fe03718f6'), description='DataStax Academy 是一个学习 Apache Cassandra 的免费资源。', title='DataStax Academy')[0m[32;1m[1;3m要查找电子邮件地址为 'patrick@datastax.com' 的用户上传到 `langchain_agent_test` keyspace 的所有视频，我们可以按照以下步骤进行：

1. 查询 `user_credentials` 表以查找与电子邮件 'patrick@datastax.com' 相关联的 `user_id`。
2. 使用第一步获得的 `user_id` 查询 `user_videos` 表以检索用户上传的所有视频。

以下是 JSON 格式的查询路径：

```json
{
  "query_paths": [
    {
      "description": "从 user_credentials 查找 user_id，然后查询 user_videos 以获取用户上传的所有视频",
      "steps": [
        {
          "table": "user_credentials",
          "query": "SELECT user_id FROM user_credentials WHERE user_email = 'patrick@datastax.com';"
        },
        {
          "table": "user_videos",
          "query": "SELECT * FROM user_videos WHERE user_id = 522b1fe2-2e36-4cef-a667-cd4237d08b89;"
        }
      ]
    }
  ]
}
```

按照这个查询路径，我们发现用户的 user_id `522b1fe2-2e36-4cef-a667-cd4237d08b89` 至少上传了一部视频，标题为 'DataStax Academy'，描述为 'DataStax Academy 是一个学习 Apache Cassandra 的免费资源。' 该视频的 video_id 为 `27066014-bad7-9f58-5a30-f63fe03718f6`。如果有更多视频，可以使用相同的查询来检索它们，必要时可以增加限制。[0m

[1m> 完成链。[0m
要查找电子邮件地址为 'patrick@datastax.com' 的用户上传到 `langchain_agent_test` keyspace 的所有视频，我们可以按照以下步骤进行：

1. 查询 `user_credentials` 表以查找与电子邮件 'patrick@datastax.com' 相关联的 `user_id`。
2. 使用第一步获得的 `user_id` 查询 `user_videos` 表以检索用户上传的所有视频。

以下是 JSON 格式的查询路径：

```json
{
  "query_paths": [
    {
      "description": "从 user_credentials 查找 user_id，然后查询 user_videos 以获取用户上传的所有视频",
      "steps": [
        {
          "table": "user_credentials",
          "query": "SELECT user_id FROM user_credentials WHERE user_email = 'patrick@datastax.com';"
        },
        {
          "table": "user_videos",
          "query": "SELECT * FROM user_videos WHERE user_id = 522b1fe2-2e36-4cef-a667-cd4237d08b89;"
        }
      ]
    }
  ]
}
```

按照这个查询路径，我们发现用户的 user_id `522b1fe2-2e36-4cef-a667-cd4237d08b89` 至少上传了一部视频，标题为 'DataStax Academy'，描述为 'DataStax Academy 是一个学习 Apache Cassandra 的免费资源。' 该视频的 video_id 为 `27066014-bad7-9f58-5a30-f63fe03718f6`。如果有更多视频，可以使用相同的查询来检索它们，必要时可以增加限制。
```