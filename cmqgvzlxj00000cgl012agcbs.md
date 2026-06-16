---
title: "¿Qué pasa realmente cuando le hablas a un LLM?"
seoTitle: "¿Qué pasa realmente cuando le hablas a un LLM?"
seoDescription: "Descubre qué pasa dentro de un LLM al procesar tu prompt. Entiende Transformers, tokens, embeddings y la arquitectura RAG con Databricks Mosaic AI.
"
datePublished: 2026-06-16T16:58:59.181Z
cuid: cmqgvzlxj00000cgl012agcbs
slug: qu-pasa-realmente-cuando-le-hablas-a-un-llm
ogImage: https://cdn.hashnode.com/uploads/og-images/689caf5773492fba4b653597/b37eaae9-8074-421f-adaa-12e1344e55ae.svg
tags: machine-learning, databricks, llm, generative-ai, rag

---

Hace unos meses decidí dar un paso importante. Tras certificarme como Analista de Datos en Databricks, me di cuenta de que el verdadero reto no está en crear dashboards bonitos, sino en entender qué pasa detrás cuando intentamos que las máquinas piensen (o al menos lo parezca). Así nace esta serie: **El Puente al Machine Learning**, un espacio donde quiero explicarte los conceptos sin rodeos matemáticos incomprensibles, pero sin caer en la trampa de usar librerías como LangChain como si fueran magia negra.

Hoy empezamos con el **Módulo 0**: lo que necesitas saber antes de escribir tu primera línea de código de IA Generativa y orquestar soluciones reales en la nube.

* * *

## ¿Qué es un Large Language Model (LLM)?

Antes de tirar código, necesitas entender qué es un LLM y por qué ha cambiado las reglas del juego.

Un **Large Language Model (LLM)** es una red neuronal gigante entrenada con enormes volúmenes de texto para adivinar la siguiente palabra en una secuencia. Suena muy básico, pero al escalar este principio a miles de millones de palabras, el modelo adquiere capacidades asombrosas: resume textos, traduce idiomas, programa código, analiza sentimientos y razona sobre problemas complejos.

El motor de todo esto es la arquitectura **Transformer**, descrita en el histórico paper *"Attention Is All You Need"* (2017). A diferencia de las antiguas redes recurrentes (RNN) que leían las oraciones palabra por palabra (lo que causaba que olvidaran el inicio del párrafo al llegar al final), el Transformer procesa toda la frase a la vez. Lo logra con el mecanismo de **Auto-Atención (Self-Attention)**, que calcula qué palabras de una oración se relacionan más con otras para definir el contexto.

<iframe src="https://norsab.github.io/Generative-AI-Engineer/Blog/figuras/figura2.html?v=9" width="100%" height="420" style="border:none;border-radius:12px;background:#0B0F19;overflow:hidden"></iframe>

* * *

## El Ecosistema Actual de Modelos

No todos los LLMs son iguales. Para la certificación de Databricks, debes distinguir claramente entre las siguientes categorías:

*   **Foundation Models (Modelos Base)**: Son los gigantes pre-entrenados con miles de millones de palabras públicas procedentes de internet, libros y código. Son generalistas capaces de realizar múltiples tareas. Ejemplos de esto son GPT-4, Claude, Gemini, LLaMA 3 y **DBRX** (el modelo abierto de Databricks).
    
*   **Fine-Tuned Models (Modelos Ajustados)**: Parten de un modelo base y reciben entrenamiento extra con un conjunto de datos muy específico. Por ejemplo, un modelo especializado en transcribir recetas médicas o generar código SQL. Mejora el rendimiento en una tarea, pero pierde habilidad en la conversación general.
    
*   **Modelos Abiertos vs. Propietarios**: Los abiertos (LLaMA 3, Mixtral, DBRX) te dan el control del código y los pesos, permitiéndote desplegarlos en tu propia infraestructura de nube. Los propietarios (GPT-4, Claude) se consumen como servicio pagado (API) y no tienes acceso a sus componentes internos.
    

### DBRX: La joya de Databricks

Para el examen, quédate con este nombre: **DBRX**. Es el modelo abierto desarrollado por Databricks en 2024. Su arquitectura es del tipo **Mixture of Experts (MoE)**. En lugar de activar todos sus parámetros en cada palabra que genera, solo activa un subconjunto de "expertos" especializados. Esto le permite competir en calidad con modelos como GPT-3.5 a una fracción del costo computacional de cómputo en la nube.

* * *

## Tokens, Ventana de Contexto y Embeddings

Tres conceptos indispensables para entender el flujo técnico del examen.

*   **Tokens**: Un LLM no lee palabras completas, las corta en trozos llamados tokens. Un token puede ser una palabra entera ("Databricks"), una parte de ella ("Data" y "bricks") o un espacio. En promedio, **1 token equivale a unas 0.75 palabras** en inglés. La facturación de las APIs se calcula en base a tokens consumidos.
    
*   **Ventana de Contexto (Context Window)**: Es el límite de memoria del modelo en una sola llamada de entrada y salida. DBRX tiene una ventana de 32K tokens, mientras que otros modelos alcanzan 128K o más. Superar este límite significa que el modelo olvidará la información más vieja.
    
*   **Embeddings**: Son representaciones matemáticas del significado de las palabras. Cada palabra o frase se convierte en una lista de números (un vector) en un espacio de muchas dimensiones. Lo valioso de esto es que palabras con significados similares (ejemplo: "médico" y "doctor") quedan ubicadas muy cerca unas de otras en este mapa matemático, facilitando las búsquedas semánticas.
    

<iframe src="https://norsab.github.io/Generative-AI-Engineer/Blog/figuras/figura1.html?v=10" width="100%" height="750" style="border:none;border-radius:12px;background:#0B0F19;overflow:hidden"></iframe>

* * *

## Casos de Uso Empresariales de GenAI

En el mundo corporativo, la IA generativa va mucho más allá de un chat simple. Los ingenieros implementamos soluciones integradas:

1.  **Asistentes de Conocimiento Interno**: Permiten a los empleados realizar consultas sobre manuales, políticas y wikis de la empresa en lenguaje natural, recuperando la respuesta correcta al instante.
    
2.  **Generación de Código y SQL**: Traducen preguntas de negocio a consultas de bases de datos. Un ejemplo de esto en Databricks es **AI/BI Genie**, que permite a los usuarios hacer analítica de datos sin saber programar SQL.
    
3.  **Extracción y Clasificación de Documentos**: Automatizan la lectura de miles de facturas o contratos para extraer campos clave (montos, fechas) o categorizarlos.
    
4.  **Agentes Autónomos**: Programas que no solo conversan, sino que ejecutan acciones como enviar un correo, agendar una reunión o actualizar un registro llamando a APIs externas.
    

* * *

## Limitaciones que Debes Conocer

No entender los límites de la IA genera aplicaciones inestables en producción. Debes dominar estos fallos típicos:

*   **Alucinaciones**: El modelo genera datos erróneos pero redactados con total seguridad. Ocurre porque el modelo es probabilístico: predice la palabra más coherente sintácticamente, no la verdad factual.
    
*   **Sesgo (Bias)**: Refleja los prejuicios o disparidades presentes en los textos de internet con los que fue entrenado.
    
*   **Límites de Contexto**: Aunque metas documentos gigantescos en el prompt, los modelos sufren del efecto **Lost in the Middle** (se pierden en el medio). Recuerdan bien el inicio y el final de las instrucciones, pero ignoran la parte central.
    
*   **Falta de Razonamiento Real**: Los modelos predicen basándose en estadísticas, no comprenden ni razonan a nivel humano. Falla en matemáticas estrictas o datos fuera de su fecha de entrenamiento.
    

* * *

## Mosaic AI: El Ecosistema GenAI de Databricks

Databricks engloba todo su desarrollo de IA generativa bajo la marca **Mosaic AI**. Estos son los servicios que integran la plataforma y que debes dominar para la certificación:

*   **Foundation Model APIs**: Permite consultar modelos abiertos (como DBRX, LLaMA) o propietarios externos (GPT-4, Claude) usando una sola API unificada. Esto facilita cambiar de modelo cambiando un parámetro en tu código.
    
*   **Model Serving**: Aloja tus modelos en producción mediante endpoints automáticos (serverless) con auto-escalado y control de costos.
    
*   **Vector Search**: Base de datos vectorial integrada directamente en Unity Catalog que se sincroniza automáticamente con tus tablas de Delta Lake.
    
*   **Mosaic AI Agent Framework**: Kit de desarrollo para programar, evaluar y desplegar agentes compuestos que usan herramientas y bases de datos.
    
*   **MLflow para GenAI**: Extensión del popular orquestador MLflow para registrar prompts, comparar respuestas, habilitar auditorías y evaluar modelos con la metodología LLM-as-a-Judge.
    

* * *

## Pipeline Típico de una Aplicación GenAI

Para estructurar una aplicación que no alucine y que conozca los datos internos de tu empresa, usamos la arquitectura **RAG (Retrieval-Augmented Generation)**. A continuación, puedes ver cómo se orquesta este flujo paso a paso:

<iframe src="https://norsab.github.io/Generative-AI-Engineer/Blog/figuras/figura3.html?v=11" width="100%" height="230" style="border:none;border-radius:12px;background:#0B0F19;overflow:hidden"></iframe>

### El Flujo de Datos en RAG Paso a Paso:

1.  **Pregunta del Usuario (Query):** Escribes tu duda en lenguaje natural (ej. *"¿Cuál es la política de viáticos?"*). Esta pregunta se vectoriza al vuelo usando un modelo de embeddings.
    
2.  **Búsqueda Vectorial (Vector Search):** El vector de la pregunta se cruza con los índices en la base de datos de vectores (gobernada por **Unity Catalog** en Databricks) para traer los fragmentos de texto más parecidos semánticamente.
    
3.  **Prompt Enriquecido (Contexto):** Esos fragmentos recuperados se inyectan directamente dentro del prompt original como contexto extra, dándole al modelo información real e interna.
    
4.  **Generación / Inferencia LLM:** El LLM lee el prompt con su contexto y genera la respuesta mediante **Mosaic AI Serving**. Al basarse estrictamente en el texto provisto, se evitan las alucinaciones.
    

### La Evolución de RAG: Classic, Graph y Agentic

No todos los flujos RAG se diseñan igual. A medida que las aplicaciones empresariales de IA se vuelven más robustas, las organizaciones evolucionan a través de tres niveles de recuperación de datos:

*   **Classic RAG (Búsqueda Secuencial):** Es el enfoque básico e inicial. La consulta del usuario se vectoriza, se realiza una búsqueda de similitud en la base de datos de vectores (**Mosaic AI Vector Search**) y se le entregan los fragmentos de texto al LLM en un único paso lineal.
    
*   **Graph RAG (Búsqueda en Grafos):** Sirve para conectar ideas sueltas en distintas fuentes. En lugar de buscar trozos de texto aislados, se extraen entidades y relaciones semánticas y se organizan en un Grafo de Conocimiento (Knowledge Graph) estructurado sobre tablas de Delta Lake.
    
*   **Agentic RAG (Agente Autónomo):** Es el nivel más avanzado y flexible. Le da el control a un agente de razonamiento diseñado con **Mosaic AI Agent Framework**. El agente evalúa la consulta, decide qué herramientas usar (Vector DB, APIs externas, búsquedas web) y valida las respuestas de forma autónoma en un ciclo de **Auto-Evaluación (Self-Correction Loop)** antes de entregarlas.
    

Explora esta comparativa visual de los tres flujos de trabajo:

<iframe src="https://norsab.github.io/Generative-AI-Engineer/Blog/figuras/figura4.html?v=13" width="100%" height="870" style="border:none;border-radius:12px;background:#0B0F19;overflow:hidden"></iframe>

* * *

## Databricks vs. El Resto: Posicionamiento en el Mercado

¿Por qué usar Databricks en lugar de conectar OpenAI directamente? El examen evalúa decisiones de diseño basadas en estas ventajas:

*   **Integración del Dato (Data-Centric AI)**: En otras plataformas debes contratar una base de datos vectorial externa, un orquestador y un servicio de hosting. En Databricks, tus datos gobernados en Delta Lake se conectan a tu base de datos vectorial (Vector Search) y a tus APIs de modelos en el mismo workspace.
    
*   **Gobernanza Unificada con Unity Catalog**: Puedes auditar quién consulta una tabla, qué modelo consume esa tabla y qué empleado llama al endpoint del modelo, todo bajo un único sistema de seguridad empresarial.
    
*   **Estándares Abiertos**: Databricks promueve el uso de tecnologías de código abierto (Delta Lake, MLflow, Spark), evitando el secuestro de proveedor (vendor lock-in).
    

* * *

## Tips de Examen: Conceptos Clave del Módulo 0

Para asegurar el éxito en la certificación, ten claros estos puntos de diseño y arquitectura en el examen:

*   **DBRX y Mixture of Experts (MoE):** Es un modelo abierto y desagregado. En lugar de procesar cada token con todos sus parámetros, DBRX activa únicamente un subconjunto de "expertos" especializados por palabra. Esto reduce significativamente los tiempos de cómputo y el coste por token en Mosaic AI Model Serving, manteniendo una precisión similar a modelos densos mucho más grandes.
    
*   **El efecto "Lost in the Middle":** Aunque incrementemos la ventana de contexto de un modelo (por ejemplo, a 32K o 128K tokens), la atención del LLM decae en la zona media de la entrada. Los datos cruciales para la consulta deben colocarse al principio o al final del prompt, o filtrarse rigurosamente antes mediante Vector Search para evitar que se pasen por alto.
    
*   **Cálculo de Tokens:** Recuerda que la correspondencia habitual en inglés es de aproximadamente **1 token ≈ 0.75 palabras** (o 100 tokens ≈ 75 palabras). Este factor es vital al estimar costes de API, límites de cuotas y la longitud total permitida en tu ventana de contexto.
    
*   **Flujo RAG de Extremo a Extremo en Databricks:** La arquitectura RAG empresarial en Databricks se integra de forma nativa:
    
    1.  Los datos sin procesar se almacenan y actualizan en **Delta Lake**.
        
    2.  Se procesan y convierten en embeddings con modelos alojados en **Model Serving**.
        
    3.  Los vectores se indexan en **Mosaic AI Vector Search**, el cual se sincroniza en tiempo real de forma automática.
        
    4.  Todo el flujo, desde las tablas origen hasta el endpoint de inferencia, queda registrado y gobernado bajo **Unity Catalog** para garantizar el cumplimiento normativo y el control de accesos.
        

¿Qué concepto de este repaso te ha parecido el más retador? ¡Cuéntame en los comentarios y sigamos construyendo este puente al Machine Learning!