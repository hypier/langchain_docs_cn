---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/docugami.ipynb
---

# Docugami
本笔记本介绍了如何从 `Docugami` 加载文档。它提供了使用该系统相对于其他数据加载器的优势。

## 前提条件
1. 安装必要的 python 包。
2. 获取您的工作区的访问令牌，并确保将其设置为 `DOCUGAMI_API_KEY` 环境变量。
3. 获取一些文档集和文档 ID，以便于处理文档，具体说明请参见此处：https://help.docugami.com/home/docugami-api


```python
# You need the dgml-utils package to use the DocugamiLoader (run pip install directly without "poetry run" if you are not using poetry)
!poetry run pip install docugami-langchain dgml-utils==0.3.0 --upgrade --quiet
```

## 快速入门

1. 创建一个 [Docugami 工作区](http://www.docugami.com)（提供免费试用）
2. 添加您的文档（PDF、DOCX 或 DOC），并允许 Docugami 处理并将其聚类为类似文档的集合，例如 NDA、租赁协议和服务协议。系统支持的文档类型没有固定的集合，创建的聚类取决于您的特定文档，您可以稍后 [更改文档集分配](https://help.docugami.com/home/working-with-the-doc-sets-view)。
3. 通过开发者游乐场为您的工作区创建一个访问令牌。 [详细说明](https://help.docugami.com/home/docugami-api)
4. 探索 [Docugami API](https://api-docs.docugami.com)，以获取您处理的文档集 ID 列表，或仅获取特定文档集的文档 ID。
6. 使用下面详细说明的 DocugamiLoader，以获取您文档的丰富语义块。
7. 可选地，构建并发布一个或多个 [报告或摘要](https://help.docugami.com/home/reports)。这有助于 Docugami 根据您的偏好改进语义 XML，添加更好的标签，然后将其作为元数据添加到 DocugamiLoader 输出中。使用 [自查询检索器](/docs/how_to/self_query) 等技术进行高准确度的文档 QA。

## 优势与其他分块技术的比较

对文档进行适当的分块对于文档检索至关重要。存在许多分块技术，包括依赖空格的简单方法和基于字符长度的递归分块。Docugami 提供了一种不同的方法：

1. **智能分块：** Docugami 将每个文档分解为一个层次化的语义 XML 树，包含不同大小的块，从单个单词或数值到整个章节。这些块遵循文档的语义轮廓，提供比任意长度或简单空格分块更有意义的表示。
2. **语义注释：** 块被注释以语义标签，这些标签在文档集之间是一致的，便于在多个文档中进行一致的层次查询，即使它们的书写和格式不同。例如，在一组租赁协议中，您可以轻松识别关键条款，如房东、租户或续租日期，以及更复杂的信息，例如任何分租条款的措辞或特定司法管辖区是否在终止条款中有例外条款。
3. **结构化表示：** 此外，XML 树指示每个文档的结构轮廓，使用表示标题、段落、列表、表格和其他常见元素的属性，并在所有支持的文档格式中一致地执行，例如扫描的 PDF 或 DOCX 文件。它适当地处理长格式文档特征，如页眉/页脚或多列流，以便进行干净的文本提取。
4. **附加元数据：** 如果用户一直在使用 Docugami，块还会附加附加元数据。这些附加元数据可用于高精度文档质量保证，而不受上下文窗口限制。请参见下面的详细代码演示。



```python
import os

from docugami_langchain.document_loaders import DocugamiLoader
```

## 加载文档

如果设置了 DOCUGAMI_API_KEY 环境变量，则无需将其显式传递给加载器，否则可以将其作为 `access_token` 参数传递。

```python
DOCUGAMI_API_KEY = os.environ.get("DOCUGAMI_API_KEY")
```

```python
docset_id = "26xpy3aes7xp"
document_ids = ["d7jqdzcj50sj", "cgd1eacfkchw"]

# 要加载给定 docset ID 中的所有文档，只需不提供 document_ids
loader = DocugamiLoader(docset_id=docset_id, document_ids=document_ids)
chunks = loader.load()
len(chunks)
```

```output
120
```

每个 `Document`（实际上是一个实际 PDF、DOC 或 DOCX 的片段）的 `metadata` 包含一些有用的附加信息：

1. **id 和 source:** 片段来源于 Docugami 中的文件（PDF、DOC 或 DOCX）的 ID 和名称。
2. **xpath:** 文档的 XML 表示中的 XPath，用于该片段。对于直接引用文档 XML 中的实际片段非常有用。
3. **structure:** 片段的结构属性，例如 h1、h2、div、table、td 等。如果调用者需要，可以用来过滤某些类型的片段。
4. **tag:** 片段的语义标签，使用各种生成和提取技术。更多细节请见： https://github.com/docugami/DFM-benchmarks

您可以通过在 `DocugamiLoader` 实例上设置以下属性来控制分片行为：

1. 您可以设置最小和最大片段大小，系统会尽量遵循这些设置，最小化截断。您可以设置 `loader.min_text_length` 和 `loader.max_text_length` 来控制这些。
2. 默认情况下，仅返回片段的文本。然而，Docugami 的 XML 知识图谱包含额外的丰富信息，包括片段内实体的语义标签。如果您希望返回的片段中包含额外的 XML 元数据，请设置 `loader.include_xml_tags = True`。
3. 此外，如果您希望 Docugami 返回其返回的片段中的父片段，可以设置 `loader.parent_hierarchy_levels`。子片段通过 `loader.parent_id_key` 值指向父片段。这在使用 [MultiVector Retriever](/docs/how_to/multi_vector) 进行 [small-to-big](https://www.youtube.com/watch?v=ihSiRrOUwmg) 检索时非常有用。请参见本笔记本后面的详细示例。

```python
loader.min_text_length = 64
loader.include_xml_tags = True
chunks = loader.load()

for chunk in chunks[:5]:
    print(chunk)
```
```output
page_content='MASTER SERVICES AGREEMENT\n <ThisServicesAgreement> This Services Agreement (the “Agreement”) sets forth terms under which <Company>MagicSoft, Inc. </Company>a <Org><USState>Washington </USState>Corporation </Org>(“Company”) located at <CompanyAddress><CompanyStreetAddress><Company>600 </Company><Company>4th Ave</Company></CompanyStreetAddress>, <Company>Seattle</Company>, <Client>WA </Client><ProvideServices>98104 </ProvideServices></CompanyAddress>shall provide services to <Client>Daltech, Inc.</Client>, a <Company><USState>Washington </USState>Corporation </Company>(the “Client”) located at <ClientAddress><ClientStreetAddress><Client>701 </Client><Client>1st St</Client></ClientStreetAddress>, <Client>Kirkland</Client>, <State>WA </State><Client>98033</Client></ClientAddress>. This Agreement is effective as of <EffectiveDate>February 15, 2021 </EffectiveDate>(“Effective Date”). </ThisServicesAgreement>' metadata={'xpath': '/dg:chunk/docset:MASTERSERVICESAGREEMENT-section/dg:chunk', 'id': 'c28554d0af5114e2b102e6fc4dcbbde5', 'name': 'Master Services Agreement - Daltech.docx', 'source': 'Master Services Agreement - Daltech.docx', 'structure': 'h1 p', 'tag': 'chunk ThisServicesAgreement', 'Liability': '', 'Workers Compensation Insurance': '$1,000,000', 'Limit': '$1,000,000', 'Commercial General Liability Insurance': '$2,000,000', 'Technology Professional Liability Errors Omissions Policy': '$5,000,000', 'Excess Liability Umbrella Coverage': '$9,000,000', 'Client': 'Daltech, Inc.', 'Services Agreement Date': 'INITIAL STATEMENT  OF WORK (SOW)  The purpose of this SOW is to describe the Software and Services that Company will initially provide to  Daltech, Inc.  the “Client”) under the terms and conditions of the  Services Agreement  entered into between the parties on  June 15, 2021', 'Completion of the Services by Company Date': 'February 15, 2022', 'Charge': 'one hundred percent (100%)', 'Company': 'MagicSoft, Inc.', 'Effective Date': 'February 15, 2021', 'Start Date': '03/15/2021', 'Scheduled Onsite Visits Are Cancelled': 'ten (10) working days', 'Limit on Liability': '', 'Liability Cap': '', 'Business Automobile Liability': 'Business Automobile Liability  covering all vehicles that Company owns, hires or leases with a limit of no less than  $1,000,000  (combined single limit for bodily injury and property damage) for each accident.', 'Contractual Liability Coverage': 'Commercial General Liability insurance including  Contractual Liability Coverage , with coverage for products liability, completed operations, property damage and bodily injury, including  death , with an aggregate limit of no less than  $2,000,000 . This policy shall name Client as an additional insured with respect to the provision of services provided under this Agreement. This policy shall include a waiver of subrogation against Client.', 'Technology Professional Liability Errors Omissions': 'Technology Professional Liability Errors & Omissions policy (which includes Cyber Risk coverage and Computer Security and Privacy Liability coverage) with a limit of no less than  $5,000,000  per occurrence and in the aggregate.'}
page_content='A. STANDARD SOFTWARE AND SERVICES AGREEMENT\n 1. Deliverables.\n Company shall provide Client with software, technical support, product management, development, and <_testRef>testing </_testRef>services (“Services”) to the Client as described on one or more Statements of Work signed by Company and Client that reference this Agreement (“SOW” or “Statement of Work”). Company shall perform Services in a prompt manner and have the final product or service (“Deliverable”) ready for Client no later than the due date specified in the applicable SOW (“Completion Date”). This due date is subject to change in accordance with the Change Order process defined in the applicable SOW. Client shall assist Company by promptly providing all information requests known or available and relevant to the Services in a timely manner.' metadata={'xpath': '/dg:chunk/docset:MASTERSERVICESAGREEMENT-section/docset:MASTERSERVICESAGREEMENT/dg:chunk[1]/docset:Standard/dg:chunk[1]', 'id': 'de60160d328df10fa2637637c803d2d4', 'name': 'Master Services Agreement - Daltech.docx', 'source': 'Master Services Agreement - Daltech.docx', 'structure': 'lim h1 lim h1 div', 'tag': 'chunk', 'Liability': '', 'Workers Compensation Insurance': '$1,000,000', 'Limit': '$1,000,000', 'Commercial General Liability Insurance': '$2,000,000', 'Technology Professional Liability Errors Omissions Policy': '$5,000,000', 'Excess Liability Umbrella Coverage': '$9,000,000', 'Client': 'Daltech, Inc.', 'Services Agreement Date': 'INITIAL STATEMENT  OF WORK (SOW)  The purpose of this SOW is to describe the Software and Services that Company will initially provide to  Daltech, Inc.  the “Client”) under the terms and conditions of the  Services Agreement  entered into between the parties on  June 15, 2021', 'Completion of the Services by Company Date': 'February 15, 2022', 'Charge': 'one hundred percent (100%)', 'Company': 'MagicSoft, Inc.', 'Effective Date': 'February 15, 2021', 'Start Date': '03/15/2021', 'Scheduled Onsite Visits Are Cancelled': 'ten (10) working days', 'Limit on Liability': '', 'Liability Cap': '', 'Business Automobile Liability': 'Business Automobile Liability  covering all vehicles that Company owns, hires or leases with a limit of no less than  $1,000,000  (combined single limit for bodily injury and property damage) for each accident.', 'Contractual Liability Coverage': 'Commercial General Liability insurance including  Contractual Liability Coverage , with coverage for products liability, completed operations, property damage and bodily injury, including  death , with an aggregate limit of no less than  $2,000,000 . This policy shall name Client as an additional insured with respect to the provision of services provided under this Agreement. This policy shall include a waiver of subrogation against Client.', 'Technology Professional Liability Errors Omissions': 'Technology Professional Liability Errors & Omissions policy (which includes Cyber Risk coverage and Computer Security and Privacy Liability coverage) with a limit of no less than  $5,000,000  per occurrence and in the aggregate.'}
page_content='2. Onsite Services.\n 2.1 Onsite visits will be charged on a <Frequency>daily </Frequency>basis (minimum <OnsiteVisits>8 hours</OnsiteVisits>).' metadata={'xpath': '/dg:chunk/docset:MASTERSERVICESAGREEMENT-section/docset:MASTERSERVICESAGREEMENT/dg:chunk[1]/docset:Standard/dg:chunk[3]/dg:chunk[1]', 'id': 'db18315b437ac2de6b555d2d8ef8f893', 'name': 'Master Services Agreement - Daltech.docx', 'source': 'Master Services Agreement - Daltech.docx', 'structure': 'lim h1 lim p', 'tag': 'chunk', 'Liability': '', 'Workers Compensation Insurance': '$1,000,000', 'Limit': '$1,000,000', 'Commercial General Liability Insurance': '$2,000,000', 'Technology Professional Liability Errors Omissions Policy': '$5,000,000', 'Excess Liability Umbrella Coverage': '$9,000,000', 'Client': 'Daltech, Inc.', 'Services Agreement Date': 'INITIAL STATEMENT  OF WORK (SOW)  The purpose of this SOW is to describe the Software and Services that Company will initially provide to  Daltech, Inc.  the “Client”) under the terms and conditions of the  Services Agreement  entered into between the parties on  June 15, 2021', 'Completion of the Services by Company Date': 'February 15, 2022', 'Charge': 'one hundred percent (100%)', 'Company': 'MagicSoft, Inc.', 'Effective Date': 'February 15, 2021', 'Start Date': '03/15/2021', 'Scheduled Onsite Visits Are Cancelled': 'ten (10) working days', 'Limit on Liability': '', 'Liability Cap': '', 'Business Automobile Liability': 'Business Automobile Liability  covering all vehicles that Company owns, hires or leases with a limit of no less than  $1,000,000  (combined single limit for bodily injury and property damage) for each accident.', 'Contractual Liability Coverage': 'Commercial General Liability insurance including  Contractual Liability Coverage , with coverage for products liability, completed operations, property damage and bodily injury, including  death , with an aggregate limit of no less than  $2,000,000 . This policy shall name Client as an additional insured with respect to the provision of services provided under this Agreement. This policy shall include a waiver of subrogation against Client.', 'Technology Professional Liability Errors Omissions': 'Technology Professional Liability Errors & Omissions policy (which includes Cyber Risk coverage and Computer Security and Privacy Liability coverage) with a limit of no less than  $5,000,000  per occurrence and in the aggregate.'}
page_content='2.2 <Expenses>Time and expenses will be charged based on actuals unless otherwise described in an Order Form or accompanying SOW. </Expenses>' metadata={'xpath': '/dg:chunk/docset:MASTERSERVICESAGREEMENT-section/docset:MASTERSERVICESAGREEMENT/dg:chunk[1]/docset:Standard/dg:chunk[3]/dg:chunk[2]/docset:ADailyBasis/dg:chunk[2]/dg:chunk', 'id': '506220fa472d5c48c8ee3db78c1122c1', 'name': 'Master Services Agreement - Daltech.docx', 'source': 'Master Services Agreement - Daltech.docx', 'structure': 'lim p', 'tag': 'chunk Expenses', 'Liability': '', 'Workers Compensation Insurance': '$1,000,000', 'Limit': '$1,000,000', 'Commercial General Liability Insurance': '$2,000,000', 'Technology Professional Liability Errors Omissions Policy': '$5,000,000', 'Excess Liability Umbrella Coverage': '$9,000,000', 'Client': 'Daltech, Inc.', 'Services Agreement Date': 'INITIAL STATEMENT  OF WORK (SOW)  The purpose of this SOW is to describe the Software and Services that Company will initially provide to  Daltech, Inc.  the “Client”) under the terms and conditions of the  Services Agreement  entered into between the parties on  June 15, 2021', 'Completion of the Services by Company Date': 'February 15, 2022', 'Charge': 'one hundred percent (100%)', 'Company': 'MagicSoft, Inc.', 'Effective Date': 'February 15, 2021', 'Start Date': '03/15/2021', 'Scheduled Onsite Visits Are Cancelled': 'ten (10) working days', 'Limit on Liability': '', 'Liability Cap': '', 'Business Automobile Liability': 'Business Automobile Liability  covering all vehicles that Company owns, hires or leases with a limit of no less than  $1,000,000  (combined single limit for bodily injury and property damage) for each accident.', 'Contractual Liability Coverage': 'Commercial General Liability insurance including  Contractual Liability Coverage , with coverage for products liability, completed operations, property damage and bodily injury, including  death , with an aggregate limit of no less than  $2,000,000 . This policy shall name Client as an additional insured with respect to the provision of services provided under this Agreement. This policy shall include a waiver of subrogation against Client.', 'Technology Professional Liability Errors Omissions': 'Technology Professional Liability Errors & Omissions policy (which includes Cyber Risk coverage and Computer Security and Privacy Liability coverage) with a limit of no less than  $5,000,000  per occurrence and in the aggregate.'}
page_content='2.3 <RegularWorkingHours>All work will be executed during regular working hours <RegularWorkingHours>Monday</RegularWorkingHours>-<Weekday>Friday </Weekday><RegularWorkingHours><RegularWorkingHours>0800</RegularWorkingHours>-<Number>1900</Number></RegularWorkingHours>. For work outside of these hours on weekdays, Company will charge <Charge>one hundred percent (100%) </Charge>of the regular hourly rate and <Charge>two hundred percent (200%) </Charge>for Saturdays, Sundays and public holidays applicable to Company. </RegularWorkingHours>' metadata={'xpath': '/dg:chunk/docset:MASTERSERVICESAGREEMENT-section/docset:MASTERSERVICESAGREEMENT/dg:chunk[1]/docset:Standard/dg:chunk[3]/dg:chunk[2]/docset:ADailyBasis/dg:chunk[3]/dg:chunk', 'id': 'dac7a3ded61b5c4f3e59771243ea46c1', 'name': 'Master Services Agreement - Daltech.docx', 'source': 'Master Services Agreement - Daltech.docx', 'structure': 'lim p', 'tag': 'chunk RegularWorkingHours', 'Liability': '', 'Workers Compensation Insurance': '$1,000,000', 'Limit': '$1,000,000', 'Commercial General Liability Insurance': '$2,000,000', 'Technology Professional Liability Errors Omissions Policy': '$5,000,000', 'Excess Liability Umbrella Coverage': '$9,000,000', 'Client': 'Daltech, Inc.', 'Services Agreement Date': 'INITIAL STATEMENT  OF WORK (SOW)  The purpose of this SOW is to describe the Software and Services that Company will initially provide to  Daltech, Inc.  the “Client”) under the terms and conditions of the  Services Agreement  entered into between the parties on  June 15, 2021', 'Completion of the Services by Company Date': 'February 15, 2022', 'Charge': 'one hundred percent (100%)', 'Company': 'MagicSoft, Inc.', 'Effective Date': 'February 15, 2021', 'Start Date': '03/15/2021', 'Scheduled Onsite Visits Are Cancelled': 'ten (10) working days', 'Limit on Liability': '', 'Liability Cap': '', 'Business Automobile Liability': 'Business Automobile Liability  covering all vehicles that Company owns, hires or leases with a limit of no less than  $1,000,000  (combined single limit for bodily injury and property damage) for each accident.', 'Contractual Liability Coverage': 'Commercial General Liability insurance including  Contractual Liability Coverage , with coverage for products liability, completed operations, property damage and bodily injury, including  death , with an aggregate limit of no less than  $2,000,000 . This policy shall name Client as an additional insured with respect to the provision of services provided under this Agreement. This policy shall include a waiver of subrogation against Client.', 'Technology Professional Liability Errors Omissions': 'Technology Professional Liability Errors & Omissions policy (which includes Cyber Risk coverage and Computer Security and Privacy Liability coverage) with a limit of no less than  $5,000,000  per occurrence and in the aggregate.'}
```

## 基本使用：Docugami Loader 用于文档问答

您可以像使用标准加载器一样使用 Docugami Loader 进行多个文档的文档问答，尽管它提供了更好的分块，遵循文档的自然轮廓。关于如何做到这一点，有很多很好的教程，例如 [这个](https://www.youtube.com/watch?v=3yPBVii7Ct0)。我们可以使用相同的代码，但使用 `DocugamiLoader` 进行更好的分块，而不是直接使用基本的分割技术加载文本或 PDF 文件。

```python
!poetry run pip install --upgrade langchain-openai tiktoken langchain-chroma hnswlib
```

```python
# 对于这个示例，我们已经有一组租赁文档的处理文档集
loader = DocugamiLoader(docset_id="zo954yqy53wp")
chunks = loader.load()

# 故意去除语义元数据，以测试在没有语义元数据的情况下如何工作
for chunk in chunks:
    stripped_metadata = chunk.metadata.copy()
    for key in chunk.metadata:
        if key not in ["name", "xpath", "id", "structure"]:
            # 移除语义元数据
            del stripped_metadata[key]
    chunk.metadata = stripped_metadata

print(len(chunks))
```
```output
4674
```
加载器返回的文档已经被拆分，因此我们不需要使用文本分割器。可选地，我们可以使用每个文档上的元数据，例如结构或标签属性，进行任何我们想要的后处理。

我们将直接使用 `DocugamiLoader` 的输出，以通常的方式设置检索问答链。

```python
from langchain.chains import RetrievalQA
from langchain_chroma import Chroma
from langchain_openai import OpenAI, OpenAIEmbeddings

embedding = OpenAIEmbeddings()
vectordb = Chroma.from_documents(documents=chunks, embedding=embedding)
retriever = vectordb.as_retriever()
qa_chain = RetrievalQA.from_chain_type(
    llm=OpenAI(), chain_type="stuff", retriever=retriever, return_source_documents=True
)
```

```python
# 用示例查询尝试检索器
qa_chain("租户在其物业上可以做什么关于标识的事情？")
```

```output
{'query': '租户在其物业上可以做什么关于标识的事情？',
 'result': '租户可以在获得房东书面许可后在其物业上放置或附加标识（数字或其他形式），该许可不得无理拒绝。标识必须符合所有适用的法律、条例等的规定，租户必须在租约终止之前移除所有此类标识。',
 'source_documents': [Document(page_content='6.01 标识。租户可以在获得房东书面许可后，在物业上放置或附加标识（数字或其他形式）或其他所需的识别标识，该许可不得无理拒绝。租户在安装或移除此类标识时造成的任何损害，租户应及时修复，费用由租户承担。任何允许的标识或其他形式的识别必须符合所有适用的法律、条例等的规定。租户还同意在搬离物业时，及时自行清除和清洁任何窗户或玻璃标识。第七条 公用设施', metadata={'id': '1c290eea05915ba0f24c4a1ffc05d6f3', 'name': '示例商业租赁/TruTone Lane 6.pdf', 'structure': 'lim h1', 'xpath': '/dg:chunk/dg:chunk/dg:chunk[2]/dg:chunk[1]/docset:TheApprovedUse/dg:chunk[12]/dg:chunk[1]'}),
  Document(page_content='6.01 标识。租户可以在获得房东书面许可后，在物业上放置或附加标识（数字或其他形式）或其他所需的识别标识，该许可不得无理拒绝。租户在安装或移除此类标识时造成的任何损害，租户应及时修复，费用由租户承担。任何允许的标识或其他形式的识别必须符合所有适用的法律、条例等的规定。租户还同意在搬离物业时，及时自行清除和清洁任何窗户或玻璃标识。第七条 公用设施', metadata={'id': '1c290eea05915ba0f24c4a1ffc05d6f3', 'name': '示例商业租赁/TruTone Lane 2.pdf', 'structure': 'lim h1', 'xpath': '/dg:chunk/dg:chunk/dg:chunk[2]/dg:chunk[1]/docset:TheApprovedUse/dg:chunk[12]/dg:chunk[1]'}),
  Document(page_content='租户可以在获得房东书面许可后，在物业上放置或附加标识（数字或其他形式）或其他所需的识别标识，该许可不得无理拒绝。租户在安装或移除此类标识时造成的任何损害，租户应及时修复，费用由租户承担。任何允许的标识或其他形式的识别必须符合所有适用的法律、条例等的规定。租户还同意在搬离物业时，及时自行清除和清洁任何窗户或玻璃标识。', metadata={'id': '58d268162ecc36d8633b7bc364afcb8c', 'name': '示例商业租赁/TruTone Lane 2.docx', 'structure': 'div', 'xpath': '/docset:OFFICELEASEAGREEMENT-section/docset:OFFICELEASEAGREEMENT/dg:chunk/docset:ARTICLEVISIGNAGE-section/docset:ARTICLEVISIGNAGE/docset:_601Signage'}),
  Document(page_content='8. 标识：\n租户不得在未经房东事先书面批准的情况下在物业上安装标识，该批准不得无理拒绝或延迟，任何此类标识应遵循任何适用的政府法律、条例、法规和其他要求。租户应在租约终止时移除所有此类标识。此类安装和移除应以避免对建筑物和其他改善造成损害或污损的方式进行，租户应修复任何损害或污损，包括但不限于因此类安装和/或移除造成的变色。', metadata={'id': '6b7d88f0c979c65d5db088fc177fa81f', 'name': '租赁协议/Bioplex, Inc.pdf', 'structure': 'lim h1 div', 'xpath': '/dg:chunk/docset:WITNESSETH-section/docset:WITNESSETH/dg:chunk/docset:TheObligation/dg:chunk[8]/dg:chunk'})]}
```

## 使用 Docugami 知识图谱进行高精度文档问答

大型文档的一个问题是，您问题的正确答案可能依赖于文档中相距较远的部分。即使使用重叠的典型分块技术，也很难为 LLM 提供足够的上下文来回答此类问题。随着即将推出的超大上下文 LLM，可能可以将大量标记，甚至整个文档，放入上下文中，但在处理非常长的文档或大量文档时，这仍然会遇到限制。

例如，如果我们提出一个更复杂的问题，需要 LLM 从文档的不同部分提取信息，即使是 OpenAI 强大的 LLM 也无法正确回答。

```python
chain_response = qa_chain("What is rentable area for the property owned by DHA Group?")
chain_response["result"]  # correct answer should be 13,500 sq ft
```

```output
" I don't know."
```

```python
chain_response["source_documents"]
```

```output
[Document(page_content='1.6 Rentable Area of the Premises.', metadata={'id': '5b39a1ae84d51682328dca1467be211f', 'name': 'Sample Commercial Leases/Shorebucks LLC_WA.pdf', 'structure': 'lim h1', 'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE-section/docset:OFFICELEASE/docset:WITNESSETH-section/docset:WITNESSETH/dg:chunk/dg:chunk/docset:BasicLeaseInformation/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS-section/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS/docset:CatalystGroup/dg:chunk[6]/dg:chunk'}),
 Document(page_content='1.6 Rentable Area of the Premises.', metadata={'id': '5b39a1ae84d51682328dca1467be211f', 'name': 'Sample Commercial Leases/Shorebucks LLC_AZ.pdf', 'structure': 'lim h1', 'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE-section/docset:WITNESSETH-section/docset:WITNESSETH/dg:chunk/dg:chunk/docset:BasicLeaseInformation/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS-section/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS/docset:MenloGroup/dg:chunk[6]/dg:chunk'}),
 Document(page_content='1.6 Rentable Area of the Premises.', metadata={'id': '5b39a1ae84d51682328dca1467be211f', 'name': 'Sample Commercial Leases/Shorebucks LLC_FL.pdf', 'structure': 'lim h1', 'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE/docset:WITNESSETH-section/docset:WITNESSETH/docset:Florida-section/docset:Florida/docset:Shorebucks/dg:chunk[5]/dg:chunk'}),
 Document(page_content='1.6 Rentable Area of the Premises.', metadata={'id': '5b39a1ae84d51682328dca1467be211f', 'name': 'Sample Commercial Leases/Shorebucks LLC_TX.pdf', 'structure': 'lim h1', 'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE/docset:WITNESSETH-section/docset:WITNESSETH/dg:chunk/dg:chunk/docset:BasicLeaseInformation/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS-section/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS/docset:LandmarkLlc/dg:chunk[6]/dg:chunk'})]
```

乍一看，答案似乎合理，但实际上是错误的。如果您仔细查看此答案的源分块，您会发现文档的分块没有将房东名称和可租面积放在同一上下文中，并且产生了无关的分块，因此答案是错误的（应该是 **13,500 sq ft**）。

Docugami 可以在这里提供帮助。分块通过不同技术创建的附加元数据进行注释，如果用户已经在使用 [Docugami](https://help.docugami.com/home/reports)。更技术性的方案将稍后添加。

具体来说，让我们请求 Docugami 在其输出中返回 XML 标签以及附加元数据：

```python
loader = DocugamiLoader(docset_id="zo954yqy53wp")
loader.include_xml_tags = (
    True  # for additional semantics from the Docugami knowledge graph
)
chunks = loader.load()
print(chunks[0].metadata)
```
```output
{'xpath': '/docset:OFFICELEASE-section/dg:chunk', 'id': '47297e277e556f3ce8b570047304560b', 'name': 'Sample Commercial Leases/Shorebucks LLC_AZ.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_AZ.pdf', 'structure': 'h1 h1 p', 'tag': 'chunk Lease', 'Lease Date': 'March  29th , 2019', 'Landlord': 'Menlo Group', 'Tenant': 'Shorebucks LLC', 'Premises Address': '1564  E Broadway Rd ,  Tempe ,  Arizona  85282', 'Term of Lease': '96  full calendar months', 'Square Feet': '16,159'}
```
我们可以使用 [自查询检索器](/docs/how_to/self_query) 来提高我们的查询准确性，利用这些附加元数据：

```python
!poetry run pip install --upgrade lark --quiet
```

```python
from langchain.chains.query_constructor.schema import AttributeInfo
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain_chroma import Chroma

EXCLUDE_KEYS = ["id", "xpath", "structure"]
metadata_field_info = [
    AttributeInfo(
        name=key,
        description=f"The {key} for this chunk",
        type="string",
    )
    for key in chunks[0].metadata
    if key.lower() not in EXCLUDE_KEYS
]

document_content_description = "Contents of this chunk"
llm = OpenAI(temperature=0)

vectordb = Chroma.from_documents(documents=chunks, embedding=embedding)
retriever = SelfQueryRetriever.from_llm(
    llm, vectordb, document_content_description, metadata_field_info, verbose=True
)
qa_chain = RetrievalQA.from_chain_type(
    llm=OpenAI(),
    chain_type="stuff",
    retriever=retriever,
    return_source_documents=True,
    verbose=True,
)
```

让我们再次运行相同的问题。由于所有的分块都有携带关键信息的元数据键/值对，即使这些信息在物理上与用于生成答案的源分块相距甚远，它也返回了正确的结果。

```python
qa_chain(
    "What is rentable area for the property owned by DHA Group?"
)  # correct answer should be 13,500 sq ft
```
```output


[1m> Entering new RetrievalQA chain...[0m

[1m> Finished chain.[0m
```

```output
{'query': 'What is rentable area for the property owned by DHA Group?',
 'result': ' The rentable area of the property owned by DHA Group is 13,500 square feet.',
 'source_documents': [Document(page_content='1.6 Rentable Area of the Premises.', metadata={'Landlord': 'DHA Group', 'Lease Date': 'March  29th , 2019', 'Premises Address': '111  Bauer Dr ,  Oakland ,  New Jersey ,  07436', 'Square Feet': '13,500', 'Tenant': 'Shorebucks LLC', 'Term of Lease': '84  full calendar  months', 'id': '5b39a1ae84d51682328dca1467be211f', 'name': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'structure': 'lim h1', 'tag': 'chunk', 'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE-section/docset:OFFICELEASE/docset:WITNESSETH-section/docset:WITNESSETH/dg:chunk/dg:chunk/docset:BasicLeaseInformation/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS-section/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS/docset:DhaGroup/dg:chunk[6]/dg:chunk'}),
  Document(page_content='<RentableAreaofthePremises><SquareFeet>13,500 </SquareFeet>square feet. This square footage figure includes an add-on factor for Common Areas in the Building and has been agreed upon by the parties as final and correct and is not subject to challenge or dispute by either party. </RentableAreaofthePremises>', metadata={'Landlord': 'DHA Group', 'Lease Date': 'March  29th , 2019', 'Premises Address': '111  Bauer Dr ,  Oakland ,  New Jersey ,  07436', 'Square Feet': '13,500', 'Tenant': 'Shorebucks LLC', 'Term of Lease': '84  full calendar  months', 'id': '4c06903d087f5a83e486ee42cd702d31', 'name': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'structure': 'div', 'tag': 'RentableAreaofthePremises', 'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE-section/docset:OFFICELEASE/docset:WITNESSETH-section/docset:WITNESSETH/dg:chunk/dg:chunk/docset:BasicLeaseInformation/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS-section/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS/docset:DhaGroup/dg:chunk[6]/docset:RentableAreaofthePremises-section/docset:RentableAreaofthePremises'}),
  Document(page_content='<TheTermAnnualMarketRent>shall mean (i) for the initial Lease Year (“Year 1”) <Money>$2,239,748.00 </Money>per year (i.e., the product of the Rentable Area of the Premises multiplied by <Money>$82.00</Money>) (the “Year 1 Market Rent Hurdle”); (ii) for the Lease Year thereafter, <Percent>one hundred three percent (103%) </Percent>of the Year 1 Market Rent Hurdle, and (iii) for each Lease Year thereafter until the termination or expiration of this Lease, the Annual Market Rent Threshold shall be <AnnualMarketRentThreshold>one hundred three percent (103%) </AnnualMarketRentThreshold>of the Annual Market Rent Threshold for the immediately prior Lease Year. </TheTermAnnualMarketRent>', metadata={'Landlord': 'DHA Group', 'Lease Date': 'March  29th , 2019', 'Premises Address': '111  Bauer Dr ,  Oakland ,  New Jersey ,  07436', 'Square Feet': '13,500', 'Tenant': 'Shorebucks LLC', 'Term of Lease': '84  full calendar  months', 'id': '6b90beeadace5d4d12b25706fb48e631', 'name': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'structure': 'div', 'tag': 'TheTermAnnualMarketRent', 'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE-section/docset:OFFICELEASE/docset:WITNESSETH-section/docset:WITNESSETH/docset:GrossRentCredit-section/docset:GrossRentCredit/dg:chunk/dg:chunk/dg:chunk/dg:chunk[2]/docset:PercentageRent/dg:chunk[2]/dg:chunk[2]/docset:TenantSRevenue/dg:chunk[2]/docset:TenantSRevenue/dg:chunk[3]/docset:TheTermAnnualMarketRent-section/docset:TheTermAnnualMarketRent'}),
  Document(page_content='1.11 Percentage Rent.\n (a) <GrossRevenue><Percent>55% </Percent>of Gross Revenue to Landlord until Landlord receives Percentage Rent in an amount equal to the Annual Market Rent Hurdle (as escalated); and </GrossRevenue>', metadata={'Landlord': 'DHA Group', 'Lease Date': 'March  29th , 2019', 'Premises Address': '111  Bauer Dr ,  Oakland ,  New Jersey ,  07436', 'Square Feet': '13,500', 'Tenant': 'Shorebucks LLC', 'Term of Lease': '84  full calendar  months', 'id': 'c8bb9cbedf65a578d9db3f25f519dd3d', 'name': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'structure': 'lim h1 lim p', 'tag': 'chunk GrossRevenue', 'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE-section/docset:OFFICELEASE/docset:WITNESSETH-section/docset:WITNESSETH/docset:GrossRentCredit-section/docset:GrossRentCredit/dg:chunk/dg:chunk/dg:chunk/docset:PercentageRent/dg:chunk[1]/dg:chunk[1]'})]}
```

这次的答案是正确的，因为自查询检索器在元数据的房东属性上创建了一个过滤器，正确地过滤出专门关于DHA集团房东的文档。生成的源块都与该房东相关，这提高了答案的准确性，即使在包含正确答案的特定块中并没有直接提到房东。

# 高级主题：基于文档知识图谱层级的小到大检索

文档本质上是半结构化的，DocugamiLoader能够导航文档的语义和结构轮廓，以提供其返回的块的父块引用。这在使用[MultiVector Retriever](/docs/how_to/multi_vector)进行[小到大](https://www.youtube.com/watch?v=ihSiRrOUwmg)检索时非常有用。

要获取父块引用，您可以将`loader.parent_hierarchy_levels`设置为非零值。

```python
from typing import Dict, List

from docugami_langchain.document_loaders import DocugamiLoader
from langchain_core.documents import Document

loader = DocugamiLoader(docset_id="zo954yqy53wp")
loader.include_xml_tags = (
    True  # for additional semantics from the Docugami knowledge graph
)
loader.parent_hierarchy_levels = 3  # for expanded context
loader.max_text_length = (
    1024 * 8
)  # 8K chars are roughly 2K tokens (ref: https://help.openai.com/en/articles/4936856-what-are-tokens-and-how-to-count-them)
loader.include_project_metadata_in_doc_metadata = (
    False  # Not filtering on vector metadata, so remove to lighten the vectors
)
chunks: List[Document] = loader.load()

# build separate maps of parent and child chunks
parents_by_id: Dict[str, Document] = {}
children_by_id: Dict[str, Document] = {}
for chunk in chunks:
    chunk_id = chunk.metadata.get("id")
    parent_chunk_id = chunk.metadata.get(loader.parent_id_key)
    if not parent_chunk_id:
        # parent chunk
        parents_by_id[chunk_id] = chunk
    else:
        # child chunk
        children_by_id[chunk_id] = chunk
```

```python
# Explore some of the parent chunk relationships
for id, chunk in list(children_by_id.items())[:5]:
    parent_chunk_id = chunk.metadata.get(loader.parent_id_key)
    if parent_chunk_id:
        # child chunks have the parent chunk id set
        print(f"PARENT CHUNK {parent_chunk_id}: {parents_by_id[parent_chunk_id]}")
        print(f"CHUNK {id}: {chunk}")
```
```output
PARENT CHUNK 7df09fbfc65bb8377054808aac2d16fd: page_content='OFFICE LEASE\n THIS OFFICE LEASE\n <Lease>(the "Lease") is made and entered into as of <LeaseDate>March 29th, 2019</LeaseDate>, by and between Landlord and Tenant. "Date of this Lease" shall mean the date on which the last one of the Landlord and Tenant has signed this Lease. </Lease>\nW I T N E S S E T H\n <TheTerms> Subject to and on the terms and conditions of this Lease, Landlord leases to Tenant and Tenant hires from Landlord the Premises. </TheTerms>\n1. BASIC LEASE INFORMATION AND DEFINED TERMS.\nThe key business terms of this Lease and the defined terms used in this Lease are as follows:' metadata={'xpath': '/docset:OFFICELEASE-section/dg:chunk', 'id': '7df09fbfc65bb8377054808aac2d16fd', 'name': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'structure': 'h1 h1 p h1 p lim h1 p', 'tag': 'chunk Lease chunk TheTerms'}
CHUNK 47297e277e556f3ce8b570047304560b: page_content='OFFICE LEASE\n THIS OFFICE LEASE\n <Lease>(the "Lease") is made and entered into as of <LeaseDate>March 29th, 2019</LeaseDate>, by and between Landlord and Tenant. "Date of this Lease" shall mean the date on which the last one of the Landlord and Tenant has signed this Lease. </Lease>' metadata={'xpath': '/docset:OFFICELEASE-section/dg:chunk', 'id': '47297e277e556f3ce8b570047304560b', 'name': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_NJ.pdf', 'structure': 'h1 h1 p', 'tag': 'chunk Lease', 'doc_id': '7df09fbfc65bb8377054808aac2d16fd'}
PARENT CHUNK bb84925da3bed22c30ea1bdc173ff54f: page_content='OFFICE LEASE\n THIS OFFICE LEASE\n <Lease>(the "Lease") is made and entered into as of <LeaseDate>January 8th, 2018</LeaseDate>, by and between Landlord and Tenant. "Date of this Lease" shall mean the date on which the last one of the Landlord and Tenant has signed this Lease. </Lease>\nW I T N E S S E T H\n <TheTerms> Subject to and on the terms and conditions of this Lease, Landlord leases to Tenant and Tenant hires from Landlord the Premises. </TheTerms>\n1. BASIC LEASE INFORMATION AND DEFINED TERMS.\nThe key business terms of this Lease and the defined terms used in this Lease are as follows:\n1.1 Landlord.\n <Landlord>Catalyst Group LLC </Landlord>' metadata={'xpath': '/docset:OFFICELEASE-section/dg:chunk', 'id': 'bb84925da3bed22c30ea1bdc173ff54f', 'name': 'Sample Commercial Leases/Shorebucks LLC_WA.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_WA.pdf', 'structure': 'h1 h1 p h1 p lim h1 p lim h1 div', 'tag': 'chunk Lease chunk TheTerms chunk Landlord'}
CHUNK 2f1746cbd546d1d61a9250c50de7a7fa: page_content='W I T N E S S E T H\n <TheTerms> Subject to and on the terms and conditions of this Lease, Landlord leases to Tenant and Tenant hires from Landlord the Premises. </TheTerms>' metadata={'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE-section/docset:OFFICELEASE/docset:WITNESSETH-section/dg:chunk', 'id': '2f1746cbd546d1d61a9250c50de7a7fa', 'name': 'Sample Commercial Leases/Shorebucks LLC_WA.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_WA.pdf', 'structure': 'h1 p', 'tag': 'chunk TheTerms', 'doc_id': 'bb84925da3bed22c30ea1bdc173ff54f'}
PARENT CHUNK 0b0d765b6e504a6ba54fa76b203e62ec: page_content='OFFICE LEASE\n THIS OFFICE LEASE\n <Lease>(the "Lease") is made and entered into as of <LeaseDate>January 8th, 2018</LeaseDate>, by and between Landlord and Tenant. "Date of this Lease" shall mean the date on which the last one of the Landlord and Tenant has signed this Lease. </Lease>\nW I T N E S S E T H\n <TheTerms> Subject to and on the terms and conditions of this Lease, Landlord leases to Tenant and Tenant hires from Landlord the Premises. </TheTerms>\n1. BASIC LEASE INFORMATION AND DEFINED TERMS.\nThe key business terms of this Lease and the defined terms used in this Lease are as follows:\n1.1 Landlord.\n <Landlord>Catalyst Group LLC </Landlord>\n1.2 Tenant.\n <Tenant>Shorebucks LLC </Tenant>' metadata={'xpath': '/docset:OFFICELEASE-section/dg:chunk', 'id': '0b0d765b6e504a6ba54fa76b203e62ec', 'name': 'Sample Commercial Leases/Shorebucks LLC_WA.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_WA.pdf', 'structure': 'h1 h1 p h1 p lim h1 p lim h1 div lim h1 div', 'tag': 'chunk Lease chunk TheTerms chunk Landlord chunk Tenant'}
CHUNK b362dfe776ec5a7a66451a8c7c220b59: page_content='1. BASIC LEASE INFORMATION AND DEFINED TERMS.' metadata={'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE-section/docset:OFFICELEASE/docset:WITNESSETH-section/docset:WITNESSETH/dg:chunk/dg:chunk/docset:BasicLeaseInformation/dg:chunk', 'id': 'b362dfe776ec5a7a66451a8c7c220b59', 'name': 'Sample Commercial Leases/Shorebucks LLC_WA.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_WA.pdf', 'structure': 'lim h1', 'tag': 'chunk', 'doc_id': '0b0d765b6e504a6ba54fa76b203e62ec'}
PARENT CHUNK c942010baaf76aa4d4657769492f6edb: page_content='OFFICE LEASE\n THIS OFFICE LEASE\n <Lease>(the "Lease") is made and entered into as of <LeaseDate>January 8th, 2018</LeaseDate>, by and between Landlord and Tenant. "Date of this Lease" shall mean the date on which the last one of the Landlord and Tenant has signed this Lease. </Lease>\nW I T N E S S E T H\n <TheTerms> Subject to and on the terms and conditions of this Lease, Landlord leases to Tenant and Tenant hires from Landlord the Premises. </TheTerms>\n1. BASIC LEASE INFORMATION AND DEFINED TERMS.\nThe key business terms of this Lease and the defined terms used in this Lease are as follows:\n1.1 Landlord.\n <Landlord>Catalyst Group LLC </Landlord>\n1.2 Tenant.\n <Tenant>Shorebucks LLC </Tenant>\n1.3 Building.\n <Building>The building containing the Premises located at <PremisesAddress><PremisesStreetAddress><MainStreet>600 </MainStreet><StreetName>Main Street</StreetName></PremisesStreetAddress>, <City>Bellevue</City>, <State>WA</State>, <Premises>98004</Premises></PremisesAddress>. The Building is located within the Project. </Building>' metadata={'xpath': '/docset:OFFICELEASE-section/dg:chunk', 'id': 'c942010baaf76aa4d4657769492f6edb', 'name': 'Sample Commercial Leases/Shorebucks LLC_WA.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_WA.pdf', 'structure': 'h1 h1 p h1 p lim h1 p lim h1 div lim h1 div lim h1 div', 'tag': 'chunk Lease chunk TheTerms chunk Landlord chunk Tenant chunk Building'}
CHUNK a95971d693b7aa0f6640df1fbd18c2ba: page_content='The key business terms of this Lease and the defined terms used in this Lease are as follows:' metadata={'xpath': '/docset:OFFICELEASE-section/docset:OFFICELEASE-section/docset:OFFICELEASE/docset:WITNESSETH-section/docset:WITNESSETH/dg:chunk/dg:chunk/docset:BasicLeaseInformation/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS-section/docset:BASICLEASEINFORMATIONANDDEFINEDTERMS/dg:chunk', 'id': 'a95971d693b7aa0f6640df1fbd18c2ba', 'name': 'Sample Commercial Leases/Shorebucks LLC_WA.pdf', 'source': 'Sample Commercial Leases/Shorebucks LLC_WA.pdf', 'structure': 'p', 'tag': 'chunk', 'doc_id': 'c942010baaf76aa4d4657769492f6edb'}
PARENT CHUNK f34b649cde7fc4ae156849a56d690495: page_content='W I T N E S S E T H\n <TheTerms> Subject to and on the terms and conditions of this Lease, Landlord leases to Tenant and Tenant hires from Landlord the Premises. </TheTerms>\n1. BASIC LEASE INFORMATION AND DEFINED TERMS.\n<BASICLEASEINFORMATIONANDDEFINEDTERMS>The key business terms of this Lease and the defined terms used in this Lease are as follows: </BASICLEASEINFORMATIONANDDEFINEDTERMS>\n1.1 Landlord.\n <Landlord><Landlord>Menlo Group</Landlord>, a <USState>Delaware </USState>limited liability company authorized to transact business in <USState>Arizona</USState>. </Landlord>\n1.2 Tenant.\n <Tenant>Shorebucks LLC </Tenant>\n1.3 Building.\n <Building>The building containing the Premises located at <PremisesAddress><PremisesStreetAddress><Premises>1564 </Premises><Premises>E Broadway Rd</Premises></PremisesStreetAddress>, <City>Tempe</City>, <USState>Arizona </USState><Premises>85282</Premises></PremisesAddress>. The Building is located within the Project. </Building>\n1.4 Project.\n <Project>The parcel of land and the buildings and improvements located on such land known as Shorebucks Office <ShorebucksOfficeAddress><ShorebucksOfficeStreetAddress><ShorebucksOffice>6 </ShorebucksOffice><ShorebucksOffice6>located at <Number>1564 </Number>E Broadway Rd</ShorebucksOffice6></ShorebucksOfficeStreetAddress>, <City>Tempe</City>, <USState>Arizona </USState><Number>85282</Number></ShorebucksOfficeAddress>. The Project is legally described in EXHIBIT "A" to this Lease. </Project>' metadata={'xpath': '/dg:chunk/docset:WITNESSETH-section/dg:chunk', 'id': 'f34b649cde7fc4ae156849a56d690495', 'name': 'Sample Commercial Leases/Shorebucks LLC_AZ.docx', 'source': 'Sample Commercial Leases/Shorebucks LLC_AZ.docx', 'structure': 'h1 p lim h1 div lim h1 div lim h1 div lim h1 div lim h1 div', 'tag': 'chunk TheTerms BASICLEASEINFORMATIONANDDEFINEDTERMS chunk Landlord chunk Tenant chunk Building chunk Project'}
CHUNK 21b4d9517f7ccdc0e3a028ce5043a2a0: page_content='1.1 Landlord.\n <Landlord><Landlord>Menlo Group</Landlord>, a <USState>Delaware </USState>limited liability company authorized to transact business in <USState>Arizona</USState>. </Landlord>' metadata={'xpath': '/dg:chunk/docset:WITNESSETH-section/docset:WITNESSETH/dg:chunk[1]/dg:chunk[1]/dg:chunk/dg:chunk[2]/dg:chunk', 'id': '21b4d9517f7ccdc0e3a028ce5043a2a0', 'name': 'Sample Commercial Leases/Shorebucks LLC_AZ.docx', 'source': 'Sample Commercial Leases/Shorebucks LLC_AZ.docx', 'structure': 'lim h1 div', 'tag': 'chunk Landlord', 'doc_id': 'f34b649cde7fc4ae156849a56d690495'}
```

```python
from langchain.retrievers.multi_vector import MultiVectorRetriever, SearchType
from langchain.storage import InMemoryStore
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings

# 用于索引子块的向量存储
vectorstore = Chroma(collection_name="big2small", embedding_function=OpenAIEmbeddings())

# 父文档的存储层
store = InMemoryStore()

# 检索器（开始时为空）
retriever = MultiVectorRetriever(
    vectorstore=vectorstore,
    docstore=store,
    search_type=SearchType.mmr,  # 使用最大边际相关性搜索
    search_kwargs={"k": 2},
)

# 将子块添加到向量存储
retriever.vectorstore.add_documents(list(children_by_id.values()))

# 将父块添加到文档存储
retriever.docstore.mset(parents_by_id.items())
```

```python
# 直接查询向量存储，应该返回块
found_chunks = vectorstore.similarity_search(
    "Birch Street允许在其物业上放置什么标志？", k=2
)

for chunk in found_chunks:
    print(chunk.page_content)
    print(chunk.metadata[loader.parent_id_key])
```
```output
24. 标志。
 <SIGNS>租户不得在项目的任何部分放置标志。然而，租户可以在房东批准的地点放置一个带有其名称的标志，靠近物业的入口（费用由租户承担），并将在建筑物目录中提供其名称的单一列示（费用由房东承担），所有这些都应符合房东不时为该项目采用的标准 <Frequency>。任何更改或额外的目录列示应在空间可用的情况下提供，并按当时的建筑标准费用收取。</SIGNS>
43090337ed2409e0da24ee07e2adbe94
<TheExterior>租户同意，所有从物业外部可见的标志、遮阳篷、防护门、安全设备和其他装置均需事先获得房东的书面批准，如有要求，还需事先获得<Org>地标</Org><Landmarks>保护委员会</Landmarks>的批准，并且不得干扰或阻塞相邻商店，然而，房东不得无理拒绝租户希望安装的标志。租户同意，任何允许的标志、遮阳篷、防护门、安全设备和其他装置应由租户自行承担费用，专业制作并保持得体，并需事先获得房东的书面批准，房东不得无理拒绝、延迟或附加条件，并应遵循房东不时可能施加的合理规则和限制。租户应向房东提交拟议标志和其他装置的图纸，显示其大小、颜色、照明和整体外观，以及其固定在物业上的方式的说明。租户不得在未获得房东书面批准之前开始安装拟议的标志和其他装置。租户不得安装任何霓虹灯标志。上述标志仅应用于识别租户的业务。未经房东事先书面同意，不得对标志和其他装置进行任何更改。租户应自费获得并向房东展示租户可能需要从任何及所有城市、州及其他有管辖权的当局获得的关于该标志或其他装置的竖立、安装、维护或使用的许可证或批准证明，并应保持该标志和其他装置及其附属物处于良好状态并令房东满意，并遵循任何及所有有管辖权的公共当局的命令、法规、要求和规则。房东同意租户在附录D中描述的初始标志。</TheExterior>
54ddfc3e47f41af7e747b2bc439ea96b
```

```python
# 查询检索器，应该返回父项（使用MMR，因为上面设置为search_type）
retrieved_parent_docs = retriever.invoke(
    "Birch Street允许在其物业上放置什么标志？"
)
for chunk in retrieved_parent_docs:
    print(chunk.page_content)
    print(chunk.metadata["id"])
```
```output
21. 服务和公用事业。
 <SERVICESANDUTILITIES>房东没有义务向物业提供除乘客电梯服务之外的任何公用设施或服务。租户应对物业内使用或消耗的水、电或任何其他公用设施的所有费用负责，并应及时支付，包括与物业单独计量相关的所有费用。租户应负责物业的出口照明、应急照明和灭火器的维修和维护。租户负责内部清洁、害虫控制和废物清除服务。房东可随时更改建筑物的电力供应商。租户对房东提供的电力、HVAC或其他服务的使用不得超过房东认为的建筑物标准，无论是在电压、额定容量、使用或整体负载方面。在任何情况下，房东均不对因未能提供任何服务而造成的损害负责，任何中断或故障均不得使租户有权获得包括租金减免在内的任何救济。如果在租赁期内，项目对停车区域或建筑物有任何类型的卡访问系统，租户应按建筑标准费用向房东购买所有物业占用者的访问卡，并应遵守与停车区域和建筑物的访问相关的建筑标准条款。</SERVICESANDUTILITIES>
22. 保证金。
 <SECURITYDEPOSIT>保证金应由房东作为租户全面和忠实履行本租约的担保，包括租金的支付。租户授予房东对保证金的担保权益。保证金可以与房东的其他资金混合，房东不对保证金支付任何利息承担责任。房东可以在必要时将保证金用于弥补租户的任何违约。如果房东如此使用保证金，租户应在收到房东通知后的<Deliver>五天 </Deliver>内向房东交付所需金额，以补充保证金至其原始金额。保证金不得视为租金的预付款或任何租户违约的损害赔偿措施，也不得作为房东可能对租户提起的任何诉讼的辩护。</SECURITYDEPOSIT>
23. 政府法规。
 <GOVERNMENTALREGULATIONS>租户应自费及时遵守（并应使所有分租人和持牌人遵守）所有政府当局的法律、法规和条例，包括1990年《美国残疾人法》（<AmericanswithDisabilitiesActDate>1990 </AmericanswithDisabilitiesActDate>）及其修订版（“ADA”），以及影响项目的所有记录的契约和限制，涉及租户、其商业行为及其对物业的使用和占有，包括因租户特定使用（相对于一般办公室使用）物业或租户对物业进行的改建而对公共区域进行的任何工作。</GOVERNMENTALREGULATIONS>
24. 标志。
 <SIGNS>租户不得在项目的任何部分放置标志。然而，租户可以在房东批准的地点放置一个带有其名称的标志，靠近物业的入口（费用由租户承担），并将在建筑物目录中提供其名称的单一列示（费用由房东承担），所有这些都应符合房东不时为该项目采用的标准 <Frequency>。任何更改或额外的目录列示应在空间可用的情况下提供，并按当时的建筑标准费用收取。</SIGNS>
25. 经纪人。
 <BROKER>房东和租户各自声明并保证，他们未就物业咨询或与任何经纪人或寻找者进行谈判，除房东的经纪人和租户的经纪人外。租户应对房东因租户与本租约有关的任何房地产经纪人提出的佣金索赔进行赔偿、辩护并使房东免受损害。房东应对租户因与本租约及与租户在本租约下的权益有关的任何索赔支付给房东的任何租赁佣金进行赔偿、辩护并使租户免受损害，除了租户与房东的经纪人和租户的经纪人之间的索赔。本文条款在本租约到期或提前终止后仍然有效。</BROKER>
26. 租期结束。
 <ENDOFTERM>租户应在本租约到期或提前终止时，将物业交还给房东，保持良好状态，清扫干净，除合理磨损外。房东或租户对物业所做的所有改建应在租期到期或提前终止时成为房东的财产。在租期到期或提前终止时，租户应自费从物业中移除所有租户的个人财产、所有计算机和电信布线，以及房东通过通知指定的所有改建。租户还应修复因移除造成的物业损坏。在租期到期或提前终止后，留在物业内的任何租户财产，房东可选择不经通知视为被放弃，在这种情况下，这些物品可由房东保留作为其财产，由房东以租户的费用处置，而无需对租户或任何其他方负责或通知。</ENDOFTERM>
27. 律师费用。
 <ATTORNEYSFEES>除非本租约另有规定，在因本租约引起或以任何方式基于或与本租约有关的任何诉讼或其他争议解决程序（包括仲裁）中，胜诉方有权从败诉方收回实际的律师费用和费用，包括与根据本条款所欠费用或费用的权利或金额的诉讼费用，以及与破产、上诉或催收程序相关的费用。除房东或租户外，任何其他个人或实体均无权根据本段收取费用。此外，如果房东成为影响物业或涉及本租约或租户在本租约下的权益的任何诉讼或程序的当事方，除房东与租户之间的诉讼外，或如果房东聘请律师收取本租约下所欠的任何金额，或在未开始诉讼的情况下强制执行本租约的任何协议、条件、契约、条款或规定，则房东所产生的费用、开支和合理的律师费用及支出应由租户支付给房东。</ATTORNEYSFEES>
43090337ed2409e0da24ee07e2adbe94
<TenantsSoleCost>租户应自费负责从物业中移除和处置所有垃圾、废物和废弃物，频率为<Frequency>每日 </Frequency>。租户应确保所有垃圾、废物和废弃物在关闭前<Stored>三十（30）分钟 </Stored>存放在物业内，除非租户在法律允许的情况下，允许在前述时间后将垃圾放置在物业外，以便在<PickUp>次日早上6:00 </PickUp>之前进行收集。垃圾应放置在物业前人行道的边缘，距离建筑物主入口最远的地点，或房东可能指定的建筑物前的其他位置。</TenantsSoleCost>
<ItsSoleCost>租户应自费在物业内采用合理的勤勉，按照最佳的现行方法防止和消灭害虫、老鼠、霉菌、真菌、过敏原、<Bacterium>细菌 </Bacterium>及其他类似情况。租户应自费定期对物业进行灭虫，令房东合理满意，并应聘请持牌的灭虫公司。房东不负责物业的任何清洁、废物移除、清洁或类似服务，且如果发现本条款中描述的任何情况存在于物业内，租户无权向房东要求任何减免、抵消或抵扣。</ItsSoleCost>
42B. 人行道的使用和维护
<TheSidewalk>租户应自费保持物业前人行道18英寸的街道清洁，无垃圾、废物、废弃物、过量水、雪和冰，并应支付因未能做到这一点而产生的任何罚款、费用或开支，作为额外租金。如果租户经营人行道咖啡馆，租户应自费维护、修理和必要时更换物业前的人行道和通向地下室的金属陷阱门（如有）。租户应在使用时在任何侧门的所有侧面张贴警告标志和锥形标志，并在打开时始终在任何此类门上附加安全杆。</TheSidewalk>
<Display>在任何情况下，租户不得使用或允许使用靠近物业或物业外的任何空间进行展示、销售或任何其他类似活动；除非[1]在法律和许可的“街头集市”类型的活动中，或[<Number>2</Number>]如果当地的分区、社区委员会[如适用]和其他市政法律、规则和条例允许人行道咖啡馆的使用，并且如果是这样，则该活动应严格遵循上述所有要求和条件。在任何情况下，租户不得使用或允许使用任何广告媒介和/或扬声器和/或声音放大器和/或广播的收音机或电视，这些声音可能在物业外被听到，或不符合房东当时有效的合理规则和规定。</Display>
42C. 店面维护
 <TheBulkheadAndSecurityGate>租户同意每月或根据房东的合理要求更频繁地清洗店面，包括门面和安全门，从上到下，并在房东认为必要时对物业内的所有窗户及其他玻璃进行修理和更换。如果租户未能按照本条款维护店面，房东可以自行承担费用进行维护，并将费用作为额外租金向租户收费。</TheBulkheadAndSecurityGate>
42D. 音乐、噪音和振动
4474c92ae7ccec9184ed2fef9f072734
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)