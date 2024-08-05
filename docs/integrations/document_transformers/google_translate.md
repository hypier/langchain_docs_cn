---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_transformers/google_translate.ipynb
---

# 谷歌翻译

[谷歌翻译](https://translate.google.com/) 是由谷歌开发的多语言神经机器翻译服务，用于将文本、文档和网站从一种语言翻译成另一种语言。

`GoogleTranslateTransformer` 允许您使用 [Google Cloud Translation API](https://cloud.google.com/translate) 翻译文本和HTML。

要使用它，您需要安装 `google-cloud-translate` python 包，并创建一个启用了 [翻译 API](https://cloud.google.com/translate/docs/setup) 的谷歌云项目。该转换器使用 [高级版 (v3)](https://cloud.google.com/translate/docs/intro-to-v3)。

- [谷歌神经机器翻译](https://en.wikipedia.org/wiki/Google_Neural_Machine_Translation)
- [大规模生产环境下的机器翻译神经网络](https://blog.research.google/2016/09/a-neural-network-for-machine.html)


```python
%pip install --upgrade --quiet  google-cloud-translate
```


```python
from langchain_core.documents import Document
from langchain_google_community import GoogleTranslateTransformer
```

## 输入

这是我们要翻译的文档


```python
sample_text = """[Generated with Google Bard]
Subject: 关键业务流程更新

Date: 2023年10月27日，星期五

亲爱的团队，

我写信是为了提供我们一些关键业务流程的更新。

销售流程

我们最近实施了一种新的销售流程，旨在帮助我们达成更多交易并增加收入。新流程包括更严格的资格审查流程、更简化的提案流程，以及更有效的客户关系管理（CRM）系统。

营销流程

我们还重新设计了我们的营销流程，以专注于创建更有针对性和吸引力的内容。我们还利用更多社交媒体和付费广告来接触更广泛的受众。

客户服务流程

我们还对客户服务流程进行了一些改进。我们实施了一个新的客户支持系统，使客户更容易获得帮助。我们还雇佣了更多的客户支持代表，以减少等待时间。

总体而言，我们对改善关键业务流程所取得的进展感到非常满意。我们相信这些变化将帮助我们实现业务增长和为客户提供最佳体验的目标。

如果您对这些变化有任何问题或反馈，请随时直接与我联系。

谢谢，

Lewis Cymbal
首席执行官，Cymbal Bank
"""
```

在初始化 `GoogleTranslateTransformer` 时，您可以包含以下参数来配置请求。

- `project_id`: Google Cloud 项目 ID。
- `location`: （可选）翻译模型位置。
  - 默认: `global` 
- `model_id`: （可选）使用的翻译 [模型 ID][models]。
- `glossary_id`: （可选）使用的翻译 [词汇表 ID][glossaries]。
- `api_endpoint`: （可选）使用的 [区域端点][endpoints]。

[models]: https://cloud.google.com/translate/docs/advanced/translating-text-v3#comparing-models  
[glossaries]: https://cloud.google.com/translate/docs/advanced/glossary  
[endpoints]: https://cloud.google.com/translate/docs/advanced/endpoints


```python
documents = [Document(page_content=sample_text)]
translator = GoogleTranslateTransformer(project_id="<YOUR_PROJECT_ID>")
```

## 输出

翻译文档后，结果将作为新文档返回，其中 `page_content` 被翻译成目标语言。

您可以向 `transform_documents()` 方法提供以下关键字参数：

- `target_language_code`: [ISO 639][iso-639] 输出文档的语言代码。
    - 有关支持的语言，请参阅 [语言支持][supported-languages]。
- `source_language_code`: （可选）[ISO 639][iso-639] 输入文档的语言代码。
    - 如果未提供，将自动检测语言。
- `mime_type`: （可选）[媒体类型][media-type] 输入文本的类型。
    - 选项：`text/plain`（默认），`text/html`。

[iso-639]: https://en.wikipedia.org/wiki/ISO_639
[supported-languages]: https://cloud.google.com/translate/docs/languages
[media-type]: https://en.wikipedia.org/wiki/Media_type


```python
translated_documents = translator.transform_documents(
    documents, target_language_code="es"
)
```


```python
for doc in translated_documents:
    print(doc.metadata)
    print(doc.page_content)
```
```output
{'model': '', 'detected_language_code': 'en'}
[Generado con Google Bard]
Asunto: Actualizaciones clave de procesos comerciales

Fecha: viernes 27 de octubre de 2023

Estimado equipo,

Le escribo para brindarle una actualización sobre algunos de nuestros procesos comerciales clave.

Proceso de ventas

Recientemente implementamos un nuevo proceso de ventas que está diseñado para ayudarnos a cerrar más acuerdos y aumentar nuestros ingresos. El nuevo proceso incluye un proceso de calificación más riguroso, un proceso de propuesta más simplificado y un sistema de gestión de relaciones con el cliente (CRM) más eficaz.

Proceso de mercadeo

También hemos renovado nuestro proceso de marketing para centrarnos en crear contenido más específico y atractivo. También estamos utilizando más redes sociales y publicidad paga para llegar a una audiencia más amplia.

proceso de atención al cliente

También hemos realizado algunas mejoras en nuestro proceso de atención al cliente. Hemos implementado un nuevo sistema de atención al cliente que facilita que los clientes obtengan ayuda con sus problemas. También hemos contratado más representantes de atención al cliente para reducir los tiempos de espera.

En general, estamos muy satisfechos con el progreso que hemos logrado en la mejora de nuestros procesos comerciales clave. Creemos que estos cambios nos ayudarán a lograr nuestros objetivos de hacer crecer nuestro negocio y brindar a nuestros clientes la mejor experiencia posible.

Si tiene alguna pregunta o comentario sobre cualquiera de estos cambios, no dude en ponerse en contacto conmigo directamente.

Gracias,

Platillo Lewis
Director ejecutivo, banco de platillos
```