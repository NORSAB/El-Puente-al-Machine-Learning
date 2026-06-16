---
title: "¿Qué pasa realmente cuando le hablas a un LLM?"
seoTitle: "¿Qué pasa realmente cuando le hablas a un LLM?"
seoDescription: "Descubre qué pasa dentro de un LLM al procesar tu prompt. Entiende Transformers, tokens, embeddings y la arquitectura RAG con Databricks Mosaic AI.
"
datePublished: 2026-06-16T16:58:59.181Z
cuid: cmqgvzlxj00000cgl012agcbs
slug: qu-pasa-realmente-cuando-le-hablas-a-un-llm
cover: https://cdn.hashnode.com/uploads/covers/689caf5773492fba4b653597/3af8886e-bb7e-46c2-ad42-f54cff98bd0b.png
ogImage: https://cdn.hashnode.com/uploads/og-images/689caf5773492fba4b653597/b37eaae9-8074-421f-adaa-12e1344e55ae.svg
tags: machine-learning, databricks, llm, generative-ai, rag

---

Hace poco tomé una decisión importante en mi carrera. Después de certificarme como Data Analyst en Databricks, entendí que el verdadero desafío no consiste en armar tableros visuales atractivos. El reto de fondo es entender qué pasa en las capas internas de los modelos cuando intentamos simular razonamiento. Así empieza esta serie, a la que llamaremos Ruta GenAI Engineer. La idea es explicar conceptos complejos de forma directa, sin fórmulas pesadas ni la magia negra de librerías tipo LangChain.

Este primer artículo sirve como el Módulo 0. Repasaremos lo básico antes de escribir la primera línea de código para conectar soluciones de inteligencia artificial en la nube.

---

## Qué es un modelo de lenguaje (LLM)

Para trabajar con estas tecnologías, lo primero es entender qué hace un LLM bajo el capó. 

Un modelo de lenguaje es una red neuronal de gran escala. Fue entrenada con enormes volúmenes de texto para predecir cuál es la palabra más probable que debería seguir a una secuencia dada. Parece una tarea sencilla. Sin embargo, al escalar este método a miles de millones de parámetros, el modelo muestra habilidades que simulan razonamiento: traduce, resume, escribe código y clasifica información de manera sumamente precisa.

La pieza clave detrás de esto es la arquitectura Transformer. Se presentó en 2017 con el artículo "Attention Is All You Need". Los modelos anteriores procesaban los textos palabra por palabra, lo que hacía que olvidaran las primeras frases al llegar al final de un texto largo. El Transformer solucionó esto al procesar la secuencia entera al mismo tiempo. Utiliza un mecanismo llamado auto-atención para calcular la relación de peso y contexto entre todas las palabras de una oración.

#### Contexto A: Financiero ("El banco estaba lleno de dinero")
El término **"banco"** recibe su mayor peso de atención (95%) de **"dinero"**, fijando su significado en una entidad bancaria.

![Auto-Atención Contexto Financiero](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/figura_2_atencion_a.png)

#### Contexto B: Mobiliario ("El banco estaba pintado de verde")
El término **"banco"** recibe su mayor peso de atención de **"verde"** y **"pintado"**, definiendo su significado en un asiento.

![Auto-Atención Contexto Mobiliario](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/figura_2_atencion_b.png)

---

## Cómo clasificamos los modelos hoy

Para la certificación de Databricks, es necesario diferenciar los tipos de modelos disponibles en el mercado.

* **Modelos base (Foundation Models)**: Son los gigantes preentrenados con terabytes de información pública de internet. Son generalistas y pueden resolver múltiples tareas de forma aceptable. Aquí entran GPT-4, Claude, LLaMA y DBRX, que es la alternativa abierta de Databricks.
* **Modelos ajustados (Fine-Tuned)**: Parten de un modelo base pero reciben un entrenamiento adicional con un conjunto de datos específico. Sirven para especializar al modelo en un dominio concreto, como redactar informes médicos o generar código SQL. Suelen perder capacidad para mantener conversaciones generales a cambio de mayor precisión en su tarea.
* **Modelos abiertos frente a propietarios**: Los modelos abiertos como LLaMA o DBRX te permiten descargar sus pesos y ejecutarlos en tu propia infraestructura de nube. Los propietarios, como los de OpenAI o Anthropic, se consumen exclusivamente a través de llamadas de pago a su API y sus entrañas son secretas.

### DBRX y los modelos de expertos
Un detalle importante para el examen es comprender la estructura de DBRX. Es el modelo de Databricks lanzado en 2024 y utiliza una arquitectura llamada mezcla de expertos (MoE). En lugar de activar todos sus parámetros para generar cada palabra, el sistema activa únicamente los subconjuntos especializados para esa tarea. Esto permite obtener una precisión equivalente a modelos más grandes pero reduciendo drásticamente el costo de procesamiento en la nube.

---

## Tokens, ventana de contexto y embeddings

Estos tres conceptos son fundamentales para el diseño de infraestructura.

* **Tokens**: Los modelos no procesan palabras completas. Dividen el texto en fragmentos llamados tokens. Un token puede ser una palabra común, una sílaba o un carácter de puntuación. Para calcular costos y límites, suelo considerar que 100 tokens representan unas 75 palabras en inglés. Las tarifas de los proveedores de API se cobran con base en estas unidades.
* **Ventana de contexto**: Representa el límite de memoria del modelo en una sola interacción. DBRX cuenta con una ventana de 32,000 tokens. Si le envías un texto que supere su ventana, el modelo empezará a olvidar los fragmentos más antiguos del historial.
* **Embeddings**: Son traducciones matemáticas del significado de las palabras. Cada fragmento de texto se convierte en un vector, una lista de números en un espacio de muchas dimensiones. Lo interesante es que los términos con significados parecidos quedan ubicados en zonas cercanas dentro de este espacio matemático. Esto es lo que permite hacer búsquedas semánticas eficaces sin depender de palabras clave exactas.

![De Texto a Vectores: Tokenización y Embeddings](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/figura_1_tokenizacion.png)

---

## Aplicaciones de GenAI en las empresas

En entornos corporativos no nos limitamos a instalar un chat sencillo. Integramos los modelos en flujos de trabajo más complejos:

1. Buscadores de información interna. Permiten consultar políticas, manuales y reglamentos de la empresa haciendo preguntas en lenguaje natural y extrayendo las fuentes exactas.
2. Traductores de lenguaje natural a SQL. Habilitan la consulta de bases de datos para usuarios sin conocimientos técnicos. En Databricks, esta funcionalidad se implementa con AI/BI Genie.
3. Extracción estructurada de documentos. Automatizan el procesamiento de facturas o contratos para extraer montos, fechas y cláusulas específicas directamente a bases de datos.
4. Agentes de ejecución. Programas que además de responder preguntas pueden interactuar con sistemas externos, como programar tareas, enviar notificaciones o actualizar registros mediante APIs.

---

## Límites técnicos y fallos comunes

Ignorar las limitaciones de los modelos suele resultar en aplicaciones inestables. Estos son los problemas que debes vigilar:

* **Alucinaciones**: El modelo responde con datos falsos pero redactados con total seguridad. Esto pasa porque el sistema es probabilístico y prioriza la coherencia de la redacción sobre la precisión histórica o fáctica.
* **Sesgo**: Los modelos tienden a reflejar las tendencias o prejuicios que venían en los textos de internet con los que se entrenaron.
* **Efecto "Lost in the Middle"**: Aunque un modelo de soporte soporte ventanas de contexto muy amplias, tiende a ignorar la información ubicada en la parte media de las instrucciones. Recuerda mejor lo que está al principio y al final del prompt.
* **Ausencia de razonamiento lógico real**: Los modelos calculan probabilidades. No entienden matemáticas complejas ni lógica abstracta por sí mismos, por lo que suelen fallar en operaciones exactas si no se les guía adecuadamente.

---

## El ecosistema de Mosaic AI en Databricks

Databricks organiza sus servicios de inteligencia artificial bajo el nombre de Mosaic AI. En la certificación necesitarás dominar estas herramientas:

* **Foundation Model APIs**: Ofrece un acceso unificado tanto a modelos abiertos como a modelos propietarios a través de una API única. Esto facilita cambiar de proveedor modificando pocas líneas de código.
* **Model Serving**: Aloja tus modelos personalizados en servidores sin infraestructura gestionable (serverless), con autoescalado y control de costos.
* **Vector Search**: Una base de datos de vectores integrada con Unity Catalog que se sincroniza sola con las tablas de tu Delta Lake.
* **Mosaic AI Agent Framework**: Un conjunto de desarrollo pensado para programar, evaluar y desplegar agentes que usen bases de datos y herramientas de terceros.
* **MLflow con soporte GenAI**: Extensión para registrar las versiones de los prompts, comparar respuestas y evaluar modelos usando metodologías automáticas como el arbitraje de LLMs (LLM-as-a-Judge).

---

## Arquitectura típica de una aplicación: RAG

Para evitar alucinaciones y darle acceso a los datos privados de la compañía, implementamos la arquitectura de Generación Aumentada por Recuperación (RAG). El proceso sigue este flujo:

![Arquitectura RAG (Retrieval-Augmented Generation)](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/figura_3_rag.png)

### El flujo de RAG paso a paso:
1. Petición del usuario. Haces una consulta en lenguaje natural. Esta pregunta se convierte en un vector matemático al vuelo.
2. Recuperación. El vector de la consulta busca en la base de datos de vectores (gobernada por Unity Catalog) para extraer los fragmentos de texto más relevantes.
3. Enriquecimiento del prompt. Esos fragmentos se inyectan como contexto dentro del prompt original.
4. Generación. El modelo lee el prompt enriquecido y redacta la respuesta basándose solo en el contexto provisto, reduciendo las alucinaciones.

### Niveles de RAG: clásico, grafos y agentes

La complejidad del flujo varía según las necesidades del negocio:

* **RAG clásico**: Es lineal y secuencial. El vector de búsqueda recupera los documentos y el modelo genera la respuesta en un solo paso.
* **Graph RAG**: Utiliza grafos de conocimiento guardados en tablas Delta para conectar entidades y relaciones complejas entre distintas fuentes de datos.
* **Agentic RAG**: Es el más flexible. Utiliza un agente que evalúa la pregunta y decide de manera autónoma qué herramientas externas o bases de datos consultar, aplicando bucles de autocorrección antes de dar la respuesta final.

![Evolución de Arquitecturas RAG](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/figura_4_rag_evolucion.png)

---

## Por qué Databricks frente a otras nubes

El examen evalúa tus decisiones de arquitectura basadas en las ventajas de la plataforma:

* **Enfoque centrado en los datos**: No necesitas bases de datos de vectores externas ni servidores de hosting separados. Todo se integra en el mismo workspace de Databricks conectado a tus tablas Delta.
* **Seguridad centralizada con Unity Catalog**: Puedes auditar desde qué tabla se generó un embedding hasta qué usuario llamó al modelo, todo bajo las mismas políticas de seguridad.
* **Flexibilidad de código abierto**: Al basarse en estándares como MLflow y Delta Lake, evitas depender de un único proveedor de nube.

---

## Notas clave para el examen

* **DBRX y MoE**: Activa expertos específicos en lugar de toda la red, reduciendo el costo por token en Mosaic AI Model Serving.
* **Lost in the Middle**: Coloca la información crítica siempre al principio o al final de tus prompts de entrada.
* **Relación de tokens**: Considera que 100 tokens equivalen aproximadamente a 75 palabras en inglés para tus cálculos de costos.
* **Ciclo de RAG empresarial**: La sincronización es automática desde Delta Lake a Vector Search y todo queda bajo la gobernanza de Unity Catalog.
