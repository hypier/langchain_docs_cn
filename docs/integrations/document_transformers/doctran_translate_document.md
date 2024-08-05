---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_transformers/doctran_translate_document.ipynb
---

# Doctran: 语言翻译

通过嵌入比较文档的好处在于可以跨多种语言进行工作。“Harrison says hello”和“Harrison dice hola”在向量空间中将占据相似的位置，因为它们在语义上具有相同的含义。

然而，在对文档进行向量化之前，使用LLM来**将文档翻译成其他语言**仍然是有用的。这在用户预计以不同语言查询知识库时尤其有帮助，或者当某种语言没有可用的最先进嵌入模型时。

我们可以使用[Doctran](https://github.com/psychic-api/doctran)库来实现这一点，该库利用OpenAI的函数调用功能在语言之间翻译文档。


```python
%pip install --upgrade --quiet  doctran
```
```output
注意：您可能需要重启内核以使用更新的包。
```

```python
from langchain_community.document_transformers import DoctranTextTranslator
from langchain_core.documents import Document
```


```python
from dotenv import load_dotenv

load_dotenv()
```



```output
True
```

## Input
这是我们要翻译的文档


```python
sample_text = """[Generated with ChatGPT]

机密文件 - 仅供内部使用

日期：2023年7月1日

主题：各类主题的更新和讨论

亲爱的团队，

希望这封邮件能找到你们。在这份文件中，我想向你们提供一些重要的更新，并讨论需要我们关注的各种主题。请将此处包含的信息视为高度机密。

安全和隐私措施
作为我们持续致力于确保客户数据安全和隐私的一部分，我们在所有系统中实施了强有力的措施。我们要表扬IT部门的John Doe（邮箱：john.doe@example.com）在增强我们网络安全方面所做的努力。今后，我们恳请大家严格遵守我们的数据保护政策和指南。此外，如果您发现任何潜在的安全风险或事件，请立即向我们的专门团队报告，邮箱是security@example.com。

人力资源更新和员工福利
最近，我们欢迎了几位新团队成员，他们在各自的部门中做出了重要贡献。我想表彰Jane Smith（社会安全号码：049-45-5928）在客户服务方面的杰出表现。Jane一直以来都收到了客户的积极反馈。此外，请记住，我们的员工福利计划的开放登记期即将到来。如果您有任何问题或需要帮助，请联系我方人力资源代表Michael Johnson（电话：418-492-3850，邮箱：michael.johnson@example.com）。

市场营销举措和活动
我们的市场营销团队一直在积极开发新策略，以提高品牌知名度并推动客户参与。我们要感谢Sarah Thompson（电话：415-555-1234）在管理我们的社交媒体平台方面的卓越努力。Sarah在过去一个月中成功将我们的关注者数量增加了20%。此外，请在日历上标记即将于7月15日举行的产品发布活动。我们鼓励所有团队成员参加并支持公司这一激动人心的里程碑。

研发项目
在追求创新的过程中，我们的研发部门一直在为各种项目不懈努力。我想表彰项目负责人David Rodriguez（邮箱：david.rodriguez@example.com）的出色工作。David在我们尖端技术开发中的贡献是不可或缺的。此外，我们想提醒大家在定于7月10日举行的每月研发头脑风暴会议上分享他们对潜在新项目的想法和建议。

请将本文件中的信息视为最高机密，并确保不与未授权人员分享。如果您对讨论的主题有任何问题或疑虑，请随时直接联系我。

感谢您的关注，让我们继续携手合作，实现我们的目标。

最诚挚的问候，

Jason Fan
联合创始人兼首席执行官
Psychic
jason@psychic.dev
"""
```


```python
documents = [Document(page_content=sample_text)]
qa_translator = DoctranTextTranslator(language="spanish")
```

## 使用同步版本的输出
翻译文档后，结果将作为新文档返回，page_content 翻译成目标语言


```python
translated_document = qa_translator.transform_documents(documents)
```


```python
print(translated_document[0].page_content)
```
```output
机密文件 - 仅供内部使用

日期：2023年7月1日

主题：关于多个主题的更新和讨论

亲爱的团队，

希望这封电子邮件能让你们一切安好。在这份文件中，我想向你们提供一些重要的更新，并讨论一些需要我们关注的主题。请将此处包含的信息视为高度机密。

安全和隐私措施
作为我们持续承诺确保客户数据安全和隐私的一部分，我们在所有系统中实施了严格的措施。我们想对IT部门的John Doe（电子邮件：john.doe@example.com）在提升我们网络安全方面的辛勤工作表示赞赏。此后，我们温馨提醒大家严格遵守我们的数据保护政策和指南。此外，如果发现任何潜在的安全风险或事件，请立即向我们专门的团队报告，邮箱为security@example.com。

人力资源更新及员工福利
最近，我们欢迎了几位新团队成员，他们对各自部门做出了重要贡献。我想表彰Jane Smith（社会安全号码：049-45-5928）在客户服务方面的杰出表现。Jane一直收到客户的积极反馈。此外，请记住，我们员工福利计划的开放注册期即将到来。如果你们有任何问题或需要帮助，请联系我人力资源代表Michael Johnson（电话：418-492-3850，电子邮件：michael.johnson@example.com）。

市场营销倡议和活动
我们的市场团队一直在积极开发新的策略，以提高品牌知名度并促进客户参与。我们想感谢Sarah Thompson（电话：415-555-1234）在管理我们的社交媒体平台方面的卓越努力。Sarah在上个月成功将我们的粉丝基础增加了20%。此外，请在日历上标记即将于7月15日举行的产品发布活动。我们鼓励所有团队成员参加并支持这一激动人心的里程碑。

研发项目
在追求创新的过程中，我们的研发部门一直在不懈努力开展多个项目。我想表彰David Rodriguez（电子邮件：david.rodriguez@example.com）作为项目负责人的出色工作。David对我们尖端技术开发的贡献是至关重要的。此外，我们提醒大家在定于7月10日的每月研发头脑风暴会议上分享你们对可能新项目的想法和建议。

请对本文件中的信息保持最高机密性，并确保不与未经授权的人士分享。如果你们对讨论的主题有任何问题或疑虑，请随时与我直接联系。

感谢你的关注，让我们继续携手实现我们的目标。

此致，

Jason Fan
联合创始人兼首席执行官
Psychic
jason@psychic.dev
```

## 使用异步版本的输出

在翻译文档后，结果将作为新文档返回，其中的 page_content 翻译成目标语言。异步版本将在文档被分割成多个部分时提高性能。它还将确保以正确的顺序返回输出。

```python
import asyncio
```

```python
result = await qa_translator.atransform_documents(documents)
```

```python
print(result[0].page_content)
```
```output
Documento Confidencial - Solo para Uso Interno

Fecha: 1 de Julio de 2023

Asunto: Actualizaciones y Discusiones sobre Varios Temas

Estimado Equipo,

Espero que este correo electrónico les encuentre bien. En este documento, me gustaría proporcionarles algunas actualizaciones importantes y discutir varios temas que requieren nuestra atención. Por favor, traten la información contenida aquí como altamente confidencial.

Medidas de Seguridad y Privacidad
Como parte de nuestro compromiso continuo de garantizar la seguridad y privacidad de los datos de nuestros clientes, hemos implementado medidas sólidas en todos nuestros sistemas. Nos gustaría elogiar a John Doe (email: john.doe@example.com) del departamento de TI por su trabajo diligente en mejorar nuestra seguridad de red. En adelante, recordamos amablemente a todos que se adhieran estrictamente a nuestras políticas y pautas de protección de datos. Además, si encuentran algún riesgo o incidente de seguridad potencial, por favor repórtenlo inmediatamente a nuestro equipo dedicado en security@example.com.

Actualizaciones de Recursos Humanos y Beneficios para Empleados
Recientemente, dimos la bienvenida a varios nuevos miembros del equipo que han hecho contribuciones significativas a sus respectivos departamentos. Me gustaría reconocer a Jane Smith (SSN: 049-45-5928) por su destacado desempeño en servicio al cliente. Jane ha recibido consistentemente comentarios positivos de nuestros clientes. Además, recuerden que el período de inscripción abierta para nuestro programa de beneficios para empleados se acerca rápidamente. Si tienen alguna pregunta o requieren asistencia, por favor contacten a nuestro representante de Recursos Humanos, Michael Johnson (teléfono: 418-492-3850, email: michael.johnson@example.com).

Iniciativas y Campañas de Marketing
Nuestro equipo de marketing ha estado trabajando activamente en el desarrollo de nuevas estrategias para aumentar el conocimiento de la marca y fomentar la participación de los clientes. Nos gustaría agradecer a Sarah Thompson (teléfono: 415-555-1234) por sus esfuerzos excepcionales en la gestión de nuestras plataformas de redes sociales. Sarah ha aumentado con éxito nuestra base de seguidores en un 20% solo en el último mes. Además, marquen sus calendarios para el próximo evento de lanzamiento de productos el 15 de Julio. Animamos a todos los miembros del equipo a asistir y apoyar este emocionante hito para nuestra empresa.

Proyectos de Investigación y Desarrollo
En nuestra búsqueda de innovación, nuestro departamento de investigación y desarrollo ha estado trabajando incansablemente en varios proyectos. Me gustaría reconocer el trabajo excepcional de David Rodriguez (email: david.rodriguez@example.com) en su rol como líder de proyecto. Las contribuciones de David al desarrollo de nuestra tecnología de vanguardia han sido fundamentales. Además, recordamos a todos que compartan sus ideas y sugerencias para posibles nuevos proyectos durante nuestra sesión mensual de lluvia de ideas de I+D, programada para el 10 de Julio.

Por favor, traten la información en este documento con la máxima confidencialidad y asegúrense de que no sea compartida con personas no autorizadas. Si tienen alguna pregunta o inquietud sobre los temas discutidos, por favor no duden en comunicarse directamente conmigo.

Gracias por su atención, y sigamos trabajando juntos para alcanzar nuestros objetivos.

Saludos cordiales,

Jason Fan
Cofundador y CEO
Psychic
jason@psychic.dev
```