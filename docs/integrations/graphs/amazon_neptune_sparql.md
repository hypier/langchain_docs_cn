---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/graphs/amazon_neptune_sparql.ipynb
---

# Amazon Neptune与SPARQL

>[Amazon Neptune](https://aws.amazon.com/neptune/) 是一个高性能图形分析和无服务器数据库，具有卓越的可扩展性和可用性。
>
>本示例展示了一个QA链，查询在`Amazon Neptune`图形数据库中的[资源描述框架（RDF）](https://en.wikipedia.org/wiki/Resource_Description_Framework)数据，使用`SPARQL`查询语言并返回可读的人类响应。
>
>[SPARQL](https://en.wikipedia.org/wiki/SPARQL) 是一种用于`RDF`图的标准查询语言。

本示例使用`NeptuneRdfGraph`类连接到Neptune数据库并加载其模式。 
`NeptuneSparqlQAChain`用于将图形和LLM连接起来，以提出自然语言问题。

该笔记本演示了一个使用组织数据的示例。

运行该笔记本的要求：
- 可从该笔记本访问的Neptune 1.2.x集群
- Python 3.9或更高版本的内核
- 对于Bedrock访问，请确保IAM角色具有以下策略

```json
{
        "Action": [
            "bedrock:ListFoundationModels",
            "bedrock:InvokeModel"
        ],
        "Resource": "*",
        "Effect": "Allow"
}
```

- 用于暂存示例数据的S3桶。该桶应与Neptune位于同一账户/区域。

## 设置

### 种子 W3C 组织数据

种子 W3C 组织数据，W3C 组织本体及一些实例。

您需要在同一地区和帐户中创建一个 S3 存储桶。将 `STAGE_BUCKET` 设置为该存储桶的名称。

```python
STAGE_BUCKET = "<bucket-name>"
```

```bash
%%bash  -s "$STAGE_BUCKET"

rm -rf data
mkdir -p data
cd data
echo getting org ontology and sample org instances
wget http://www.w3.org/ns/org.ttl 
wget https://raw.githubusercontent.com/aws-samples/amazon-neptune-ontology-example-blog/main/data/example_org.ttl 

echo Copying org ttl to S3
aws s3 cp org.ttl s3://$1/org.ttl
aws s3 cp example_org.ttl s3://$1/example_org.ttl

```

批量加载 org ttl - 包括本体和实例

```python
%load -s s3://{STAGE_BUCKET} -f turtle --store-to loadres --run
```

```python
%load_status {loadres['payload']['loadId']} --errors --details
```

### 设置链

```python
!pip install --upgrade --quiet langchain langchain-community langchain-aws
```

** 重启内核 **

### 准备一个示例


```python
EXAMPLES = """

<question>
查找组织。
</question>

<sparql>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> 
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> 
PREFIX org: <http://www.w3.org/ns/org#> 

select ?org ?orgName where {{
    ?org rdfs:label ?orgName .
}} 
</sparql>

<question>
查找一个组织的站点
</question>

<sparql>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> 
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> 
PREFIX org: <http://www.w3.org/ns/org#> 

select ?org ?orgName ?siteName where {{
    ?org rdfs:label ?orgName .
    ?org org:hasSite/rdfs:label ?siteName . 
}} 
</sparql>

<question>
查找一个组织的子组织
</question>

<sparql>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> 
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> 
PREFIX org: <http://www.w3.org/ns/org#> 

select ?org ?orgName ?subName where {{
    ?org rdfs:label ?orgName .
    ?org org:hasSubOrganization/rdfs:label ?subName  .
}} 
</sparql>

<question>
查找一个组织的组织单位
</question>

<sparql>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> 
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> 
PREFIX org: <http://www.w3.org/ns/org#> 

select ?org ?orgName ?unitName where {{
    ?org rdfs:label ?orgName .
    ?org org:hasUnit/rdfs:label ?unitName . 
}} 
</sparql>

<question>
查找一个组织的成员。还要找到他们的经理或他们汇报的成员。
</question>

<sparql>
PREFIX org: <http://www.w3.org/ns/org#> 
PREFIX foaf: <http://xmlns.com/foaf/0.1/> 

select * where {{
    ?person rdf:type foaf:Person .
    ?person  org:memberOf ?org .
    OPTIONAL {{ ?person foaf:firstName ?firstName . }}
    OPTIONAL {{ ?person foaf:family_name ?lastName . }}
    OPTIONAL {{ ?person  org:reportsTo ??manager }} .
}}
</sparql>


<question>
查找一个组织的变更事件，例如合并和收购
</question>

<sparql>
PREFIX org: <http://www.w3.org/ns/org#> 

select ?event ?prop ?obj where {{
    ?org rdfs:label ?orgName .
    ?event rdf:type org:ChangeEvent .
    ?event org:originalOrganization ?origOrg .
    ?event org:resultingOrganization ?resultingOrg .
}}
</sparql>

"""
```


```python
import boto3
from langchain.chains.graph_qa.neptune_sparql import NeptuneSparqlQAChain
from langchain_aws import ChatBedrock
from langchain_community.graphs import NeptuneRdfGraph

host = "<your host>"
port = 8182  # change if different
region = "us-east-1"  # change if different
graph = NeptuneRdfGraph(host=host, port=port, use_iam_auth=True, region_name=region)

# Optionally change the schema
# elems = graph.get_schema_elements
# change elems ...
# graph.load_schema(elems)

MODEL_ID = "anthropic.claude-v2"
bedrock_client = boto3.client("bedrock-runtime")
llm = ChatBedrock(model_id=MODEL_ID, client=bedrock_client)

chain = NeptuneSparqlQAChain.from_llm(
    llm=llm,
    graph=graph,
    examples=EXAMPLES,
    verbose=True,
    top_K=10,
    return_intermediate_steps=True,
    return_direct=False,
)
```

## 提问
取决于我们上面获取的数据


```python
chain.invoke("""How many organizations are in the graph""")
```


```python
chain.invoke("""Are there any mergers or acquisitions""")
```


```python
chain.invoke("""Find organizations""")
```


```python
chain.invoke("""Find sites of MegaSystems or MegaFinancial""")
```


```python
chain.invoke("""Find a member who is manager of one or more members.""")
```


```python
chain.invoke("""Find five members and who their manager is.""")
```


```python
chain.invoke(
    """Find org units or suborganizations of The Mega Group. What are the sites of those units?"""
)
```