---
title: "Módulo 5: Evaluación de Aplicaciones GenAI con MLflow Evaluate y Mosaic AI"
seoTitle: "Evaluación GenAI con MLflow y Mosaic AI | Databricks"
seoDescription: "Métricas de recuperación, tríada RAG, evaluación con LLM-as-a-Judge y curación de Golden Datasets en Databricks Mosaic AI.
"
datePublished: 2026-09-02T22:34:02.731Z
cuid: cmtkoaxxm00000agmep5ycyop
slug: modulo-5-evaluacion-aplicaciones-genai-mlflow-mosaic-ai
cover: https://cdn.hashnode.com/uploads/covers/689caf5773492fba4b653597/e95a3188-a13b-4f36-854b-b2dc7d825bd8.svg
tags: artificial-intelligence, machine-learning, databricks, generative-ai, llmops

---


### Arquitectura de Evaluación Continua para RAG y Agentes
*Métricas cuantitativas de recuperación, auditoría semántica con LLM-as-a-Judge y diagnóstico end-to-end en el Lakehouse*

En los módulos anteriores construimos agentes de inteligencia artificial, integramos herramientas gobernadas en Unity Catalog y diseñamos flujos resilientes con Databricks Workflows. Sin embargo, al pasar de un prototipo a un sistema en producción surge una pregunta decisiva: **¿cómo medimos objetivamente si una modificación en el prompt, en los fragmentos de texto o en el modelo realmente mejoró el sistema?**

A diferencia del machine learning tradicional, donde optimizamos métricas estandarizadas como RMSE, F1-Score o AUC-ROC sobre etiquetas fijas, las aplicaciones de IA generativa devuelven lenguaje natural libre. Dos respuestas pueden redactarse de forma completamente diferente y ser igual de correctas, o sonar convincentes mientras inventan datos con absoluta soltura.

En este artículo exploraremos la metodología completa de evaluación recomendada por Databricks para la certificación **Databricks Certified Generative AI Engineer Associate**: desde métricas de ranking para búsqueda vectorial hasta la arquitectura de **LLM-as-a-Judge**, el uso de **MLflow Evaluate** y la gestión de **Golden Datasets** mediante **Mosaic AI Agent Evaluation**.

### Infografía de Arquitectura: Mosaic AI Agent Evaluation
*Mapa integral de fuentes de datos, evaluación cualitativa y bucle continuo de mejora*

![Infografía General de Evaluación GenAI](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/Modulo_5/infografia_modulo5_evaluacion_1600x840.svg)

---

## 1. El Dilema de la Evaluación en Sistemas Generativos

Evaluar aplicaciones compuestas (como pipelines RAG o grafos multi-agente) presenta tres dificultades fundamentales:

1. **La trampa de las métricas sintácticas:** Métricas heredadas del procesamiento de lenguaje natural clásico como **BLEU** o **ROUGE** comparan la coincidencia exacta de n-gramas entre la respuesta generada y una respuesta de referencia. Si el modelo responde *"El plazo límite vence el viernes"* y la referencia dice *"La fecha de entrega final es el viernes"*, BLEU asigna un puntaje bajo a pesar de que el significado es idéntico. Peor aún: una alucinación redactada con palabras similares a la referencia obtendrá un puntaje alto engañoso.
2. **Sistemas en cascada:** Cuando un agente comete un error, el fallo rara vez es atribuible únicamente al modelo de lenguaje. Puede originarse en una fragmentación deficiente de los PDFs, en un recuperador vectorial que trajo texto irrelevante, en una herramienta que devolvió parámetros vacíos o en un prompt de sistema ambiguo.
3. **El costo de la revisión humana manual:** Someter miles de respuestas al criterio de auditores humanos es lento, costoso y no escala para ciclos modernos de CI/CD.

Para superar este cuello de botella, la industria y Databricks adoptan un enfoque desacoplado: **medir cuantitativamente la calidad de la recuperación** y **auditar cualitativamente la generación mediante modelos evaluadores con rúbricas rigurosas**.

---

## 2. Métricas Cuantitativas del Recuperador (Vector Search)

En cualquier arquitectura RAG, el recuperador define el techo de calidad de la respuesta final: si el contexto necesario no entra al prompt, el modelo generador no tiene cómo responder correctamente.

### Métricas de Recuperación en Búsqueda Vectorial
*Fórmulas de evaluación, propósito y sensibilidad al ranking de fragmentos recuperados*

![Métricas Cuantitativas del Recuperador](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/Modulo_5/figura_1_metricas_recuperacion_rag.svg)

Las métricas fundamentales evaluadas en el examen oficial son:

### A. Precision@K (Precisión en los primeros K fragmentos)
Mide la proporción de fragmentos devueltos en las primeras $K$ posiciones que contienen información verdaderamente pertinente a la pregunta:

$$\text{Precision@K} = \frac{\text{Número de fragmentos relevantes en los primeros } K}{K}$$

* **Importancia práctica:** Un valor bajo de Precision@K indica que estamos inyectando ruido en la ventana de contexto. Esto encarece el consumo de tokens y expone al modelo al fenómeno de *lost in the middle*, donde la información importante se diluye entre párrafos irrelevantes.

### B. Recall@K (Sensibilidad / Cobertura)
Mide qué proporción del conocimiento total necesario para responder fue capturada dentro del conjunto de $K$ fragmentos:

$$\text{Recall@K} = \frac{\text{Número de fragmentos relevantes en los primeros } K}{\text{Total de fragmentos relevantes existentes}}$$

* **Importancia práctica:** Si el Recall@K es deficiente, la respuesta sufrirá omisiones críticas o forzará al modelo a alucinar para rellenar los vacíos lógicos.

### C. Mean Reciprocal Rank (MRR)
Evalúa qué tan arriba en la lista aparece el **primer fragmento relevante**. Para un conjunto de $Q$ preguntas de evaluación:

$$\text{MRR} = \frac{1}{|Q|} \sum_{i=1}^{|Q|} \frac{1}{\text{rank}_i}$$

Donde $\text{rank}_i$ es la posición ordinal de la primera coincidencia correcta para la consulta $i$. Si el primer acierto está en la primera posición, el puntaje es $1.0$; si aparece en la tercera posición, es $0.33$; si no aparece en el Top-K devuelto, es $0.0$.
* **Importancia práctica:** Es la métrica reina cuando el sistema solo extrae una única respuesta directa o cuando el usuario final solo lee el primer resultado mostrado.

### D. Normalized Discounted Cumulative Gain (NDCG@K)
A diferencia de Precision y Recall, que tratan la relevancia de forma binaria (sí o no), NDCG permite ponderar **grados de relevancia** (por ejemplo: $2 = \text{relevancia exacta}$, $1 = \text{parcial}$, $0 = \text{nula}$) y penaliza fuertemente que los fragmentos más relevantes aparezcan al final del bloque de contexto. Es indispensable para calibrar modelos de **Reranking** (Cross-Encoders).

---

## 3. La Tríada de RAG y Evaluación con LLM-as-a-Judge

Una vez que evaluamos la búsqueda de datos, debemos auditar el comportamiento del modelo generador. Aquí entra en juego la técnica de **LLM-as-a-Judge**: utilizar un modelo de lenguaje de alta capacidad (como DBRX Instruct o GPT-4o) actuando como árbitro imparcial regido por un prompt de evaluación estructurado.

### La Tríada Fundamental de Evaluación RAG
*Los tres pilares para diagnosticar fallas de recuperación vs. generación*

![La Tríada de Evaluación RAG](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/Modulo_5/napkin_triada_evaluacion_rag.svg)

En el marco de **Mosaic AI Agent Evaluation**, todo diagnóstico se fundamenta en la denominada **Tríada de RAG**:

1. **Context Relevance (Relevancia del Contexto):**
   * *Pregunta clave:* ¿El recuperador trajo únicamente información relevante a la consulta del usuario?
   * *Diagnóstico de fallo:* Si el puntaje es bajo, ajusta el tamaño de fragmentación (*chunk size*), el umbral de similitud o implementa un paso de reranking.
2. **Groundedness / Faithfulness (Fidelidad o Ausencia de Alucinación):**
   * *Pregunta clave:* ¿Cada afirmación realizada en la respuesta generada se desprende directamente de los fragmentos recuperados?
   * *Diagnóstico de fallo:* Si este puntaje cae, el modelo está inventando hechos (alucinando). Se corrige reforzando el prompt del sistema (*"Responde únicamente utilizando los hechos provistos en el contexto..."*) o reduciendo la temperatura a cero.
3. **Answer Relevance (Pertinencia de la Respuesta):**
   * *Pregunta clave:* ¿La respuesta resuelve la necesidad real del usuario?
   * *Diagnóstico de fallo:* Si el Groundedness es perfecto pero el Answer Relevance es bajo, el modelo suele estar dando evasivas, citando párrafos desconectados o negándose innecesariamente a responder.

### Arquitectura Técnica de LLM-as-a-Judge
*Flujo del triplete de entrada, inyección de rúbrica Chain-of-Thought y veredicto factual*

![Arquitectura LLM-as-a-Judge](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/Modulo_5/figura_2_llm_judge_mosaic_evaluation.svg)

---

## 4. Control y Mitigación de Sesgos en el Juez Evaluador

Confiar la evaluación a un modelo de lenguaje introduce riesgos metodológicos sistemáticos si no se configuran salvaguardas adecuadas:

### Flujo Metodológico del Evaluador
*Ciclo de 4 pasos para garantizar reproducibilidad en las calificaciones*

![Flujo LLM-as-a-Judge](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/Modulo_5/napkin_llm_as_judge_workflow.svg)

Al diseñar rúbricas para el examen y para producción, debemos controlar tres sesgos conocidos:

* **Sesgo de Posición (Position Bias):** En evaluaciones comparativas donde se presentan dos respuestas candidatas (A vs. B), los modelos tienden a favorecer la primera opción.
  * *Solución:* Implementar **Swap Evaluation**: evaluar primero $(A, B)$ y luego invertir el orden $(B, A)$. Solo se valida el resultado si el juicio es coherente en ambas orientaciones.
* **Sesgo de Verbosidad (Verbosity Bias):** Los modelos evaluadores tienden a premiar respuestas extensas y floridas, asumiendo erróneamente que mayor longitud implica mayor profundidad.
  * *Solución:* Especificar explícitamente en la rúbrica del juez que se penalice el relleno y se premie la concisión factual.
* **Sesgo de Auto-Preferencia (Self-Enhancement Bias):** Un modelo evaluador suele asignar calificaciones más altas a respuestas generadas por modelos de su misma familia arquitectónica.
  * *Solución:* Emplear modelos independientes de familias distintas como jueces de referencia cruzada (por ejemplo, evaluar agentes basados en Llama usando DBRX o GPT-4o).
* **Chain-of-Thought Obligatorio:** El prompt del juez debe exigir que el modelo explique su razonamiento factual paso a paso **antes** de emitir la calificación final. Forzar la generación de tokens de razonamiento incrementa sustancialmente la correlación con evaluadores humanos.

---

## 5. Automatización de Pruebas con MLflow Evaluate

Databricks integra la evaluación continua dentro del ciclo de vida de **MLflow 2.x**. Mediante la función unificada `mlflow.evaluate()`, podemos auditar modelos en memoria, grafos de LangChain o tablas estáticas de inferencia de forma automatizada.

### Pipeline Integrado de MLflow Evaluate
*Entradas, ejecución automatizada y artefactos de diagnóstico*

![Pipeline de MLflow Evaluate](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/Modulo_5/figura_3_mlflow_evaluate_workflow.svg)

A continuación se presenta un script completo y listo para producción que ilustra cómo evaluar un pipeline RAG utilizando las métricas estándar de Mosaic AI:

```python
import mlflow
import pandas as pd
from mlflow.metrics.genai import faithfulness, answer_relevance

# 1. Definir el Dataset de Prueba (Golden Evaluation Set)
eval_data = pd.DataFrame({
    "inputs": [
        "¿Cuál es la política de retención para tablas Delta en producción?",
        "¿Cómo se configuran los permisos de lectura en Unity Catalog?"
    ],
    "context": [
        "Las tablas Delta en producción mantienen un historial de 30 días con VACUUM RETAIN 720 HOURS.",
        "Para otorgar lectura en Unity Catalog se ejecuta GRANT SELECT ON TABLE catalog.schema.table TO `grupo`;"
    ],
    "ground_truth": [
        "Se retiene el historial por 30 días mediante la instrucción VACUUM con 720 horas.",
        "Se utiliza la sentencia GRANT SELECT ON TABLE indicando la ruta de tres niveles y el grupo receptor."
    ]
})

# 2. Configurar las métricas de LLM-as-a-Judge apuntando a endpoints gobernados
judge_endpoint = "endpoints:/databricks-dbrx-instruct"

custom_faithfulness = faithfulness(model=judge_endpoint)
custom_relevance = answer_relevance(model=judge_endpoint)

# 3. Función de inferencia del agente a evaluar
def agent_predict(inputs_df):
    responses = []
    for _, row in inputs_df.iterrows():
        # Simulación de respuesta generada por el agente
        query = row["inputs"]
        if "retención" in query:
            responses.append("La retención estándar para tablas Delta es de 30 días según la política corporativa.")
        else:
            responses.append("Se configuran permisos con GRANT SELECT en el catálogo correspondiente.")
    return responses

# 4. Iniciar la corrida de evaluación en MLflow
with mlflow.start_run(run_name="eval_modulo5_rag_baseline"):
    eval_dataset = mlflow.data.from_pandas(eval_data, name="golden_benchmark_v1")
    
    results = mlflow.evaluate(
        model=agent_predict,
        data=eval_data,
        targets="ground_truth",
        model_type="question-answering",
        evaluators="default",
        extra_metrics=[custom_faithfulness, custom_relevance]
    )

    print("Métricas Globales Obtenidas:")
    for metric_name, score in results.metrics.items():
        print(f" - {metric_name}: {score:.4f}")

    # Visualizar tabla de diagnósticos por cada fila
    eval_table = results.tables["eval_results_table"]
    print("\nPrimer resultado detallado con feedback CoT:")
    print(eval_table[["inputs", "outputs", "faithfulness/v1/score", "faithfulness/v1/justification"]].head(1))
```

Al concluir la corrida, MLflow registra automáticamente la **Evaluation Table** interactiva en la interfaz web de Databricks, permitiendo inspeccionar fila por fila por qué un caso reprobó la rúbrica y qué justificó el juez.

---

## 6. Curación de Golden Datasets con Mosaic AI Agent Evaluation

El rendimiento de cualquier suite de evaluación depende de la representatividad de su conjunto de datos de referencia (**Golden Dataset**). Construir este dataset desde cero de forma manual es ineficiente; por ello, Databricks propone un **bucle cerrado de calidad** alimentado directamente por el tráfico real de producción.

### Ciclo Continuo de Curación
*De las tablas de inferencia en producción a las pruebas de regresión en CI/CD*

![Ciclo Continuo de Golden Datasets](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/Modulo_5/napkin_golden_dataset_curation.svg)

El flujo se estructura en cinco etapas continuas:

1. **Inference Tables en Delta Lake:** Al desplegar un endpoint en **Mosaic AI Model Serving**, activamos las tablas de inferencia con un solo clic. El Lakehouse registra de manera asíncrona y sin penalizar la latencia del usuario cada solicitud, respuesta, latencia y cantidad de tokens en una tabla Delta gobernada por Unity Catalog.
2. **Filtrado Inteligente de Anomalías:** Mediante consultas programadas con Databricks SQL, filtramos las interacciones críticas: respuestas donde el usuario presionó el botón de *thumbs-down*, sesiones donde la latencia superó los 4 segundos o donde el clasificador de seguridad alertó sobre posibles inyecciones.
3. **Mosaic AI Review App:** Databricks proporciona una aplicación web preconstruida que permite a los expertos del dominio (abogados, auditores, analistas de negocio) revisar esas conversaciones anómalas sin tener acceso a la consola de código. El experto puede corregir la respuesta esperada y marcarla como *verificada*.
4. **Almacenamiento ACID y Time Travel:** Las preguntas y respuestas aprobadas se consolidan en la tabla del Golden Dataset en Delta Lake. Gracias al versionado nativo (*Time Travel*), cualquier experimento futuro puede ejecutarse exactamente contra la versión `VERSION AS OF 12` del dataset dorado, garantizando comparaciones rigurosas a lo largo de meses de desarrollo.
5. **Gatekeeper de CI/CD:** Antes de promover una nueva versión de un agente o actualizar el índice vectorial a producción, una pipeline automatizada en Databricks Workflows ejecuta `mlflow.evaluate`. Si la métrica de Groundedness cae por debajo del 95%, el despliegue se bloquea de forma preventiva.

---

## 7. Tips Clave y Preguntas Frecuentes para el Examen de Certificación

Para asegurar el éxito en el dominio de evaluación del examen **Databricks Certified Generative AI Engineer Associate**, ten presentes estos tips estratégicos de alta recurrencia:

### Cheat Sheet de Conceptos de Examen:

| Concepto / Métrica | Pregunta Clave de Examen | Regla Mnemotécnica / Decisión |
|---|---|---|
| **Groundedness (Faithfulness)** | ¿El modelo inventó hechos fuera del contexto? | Mide **alucinaciones**. Solo compara Respuesta vs. Contexto (ignora la Query). |
| **Answer Relevance** | ¿La respuesta atendió lo que se preguntó? | Compara Respuesta vs. Query. Puede tener Groundedness perfecto y reprobar Relevance si fue evasiva. |
| **Context Relevance** | ¿El recuperador trajo fragmentos útiles? | Compara Contexto vs. Query. Penaliza fragmentos irrelevantes que encarecen el prompt. |
| **Precision@K baja** | ¿Qué causa en el LLM? | Inyección de **ruido** en el prompt y efecto *lost in the middle*. |
| **Recall@K bajo** | ¿Qué causa en el LLM? | **Omisión** de información crítica que fuerza al modelo a alucinar. |
| **MRR** | ¿Cuándo es la métrica reina? | Cuando el sistema o usuario solo consume la **primera coincidencia** devuelta. |
| **NDCG@K** | ¿Para qué componente es vital? | Calibración y benchmarking de modelos de **Reranking** (Cross-Encoders). |
| **Inference Tables** | ¿Dónde se guardan los logs de serving? | En **tablas Delta gobernadas en Unity Catalog**, registradas asíncronamente sin latencia. |
| **Mosaic AI Review App** | ¿Quién la utiliza y para qué? | **Expertos de negocio (SMEs) sin código** para etiquetar y curar el Golden Dataset. |
| **Temperatura del Juez** | ¿Por qué siempre `temperature = 0.0`? | Para garantizar **determinismo, consistencia y reproducibilidad** en las notas emitidas. |
| **Swap Evaluation** | ¿Qué sesgo mitiga en LLM Judges? | Neutraliza el **sesgo de posición** (Position Bias) evaluando $(A, B)$ y $(B, A)$. |

---

### Escenarios Prácticos de Examen y Arquitectura:

* **1. Escenario de degradación por contexto ruidoso:**
  * *Pregunta:* Durante la auditoría de un asistente RAG, el equipo detecta que la latencia se duplicó y el LLM mezcla cláusulas contradictorias. El análisis del recuperador reporta un **Recall@5 de 1.0**, pero un **Precision@5 de 0.20**. ¿Cuál es la causa raíz del fallo y qué ajuste arquitectónico procede?
  * *Respuesta:* El recuperador tiene cobertura completa pero inyecta un 80% de fragmentos irrelevantes (ruido) en el prompt, provocando el fenómeno de *lost-in-the-middle* y disparando el costo de tokens. Se debe calibrar el umbral de similitud en Vector Search o incorporar un paso de **Reranking con Cross-Encoder** para filtrar chunks espurios antes de construir el contexto.

* **2. Desacoplamiento de fallos en la Tríada RAG:**
  * *Pregunta:* Un bot de atención al cliente obtiene una calificación perfecta de **Groundedness = 1.0**, pero los usuarios reclaman reiteradamente que el sistema "no resuelve lo que se le pide". ¿Qué métrica de Mosaic AI Agent Evaluation está fallando y cómo se corrige?
  * *Respuesta:* La métrica deficiente es **Answer Relevance**. El modelo no alucina (todo lo que dice está en los documentos), pero genera respuestas evasivas, incompletas o cita políticas sin contestar la consulta puntual. Se resuelve reforzando el *system prompt* del generador para priorizar síntesis directa y completitud sobre la intención del usuario.

* **3. Auditoría de producción sin penalización de latencia:**
  * *Pregunta:* Una entidad financiera requiere registrar el 100% de las solicitudes, respuestas, latencias y llamadas a herramientas de sus agentes en producción para análisis de compliance, pero prohíbe terminantemente agregar overhead o latencia síncrona en el endpoint. ¿Qué mecanismo nativo de Databricks debe habilitarse?
  * *Respuesta:* Activar **Inference Tables** en el endpoint de Mosaic AI Model Serving. El Lakehouse captura el tráfico de forma asíncrona y lo persiste automáticamente en una tabla Delta gobernada bajo Unity Catalog, habilitando auditorías SQL y Time Travel sin impactar el SLA del cliente.

* **4. Neutralización sistemática del sesgo de posición:**
  * *Pregunta:* Al utilizar un LLM-as-a-Judge para comparar dos arquitecturas de agentes (Modelo A vs. Modelo B), se detecta que el juez casi siempre califica mejor a la primera opción que lee. ¿Qué técnica metodológica debe implementarse para mitigar este sesgo?
  * *Respuesta:* **Swap Evaluation (Evaluación por Permutación)**. El pipeline evalúa el par de respuestas en ambas orientaciones: primero $(A, B)$ y luego $(B, A)$. Si el juez no mantiene un veredicto coherente tras la inversión, la comparación se declara empate o se promedian los puntajes numéricos para eliminar el sesgo de posición (*Position Bias*).

* **5. Curación de datasets dorados por revisores no técnicos:**
  * *Pregunta:* Los auditores legales de la empresa necesitan inspeccionar conversaciones anómalas de producción y corregir la respuesta esperada para sumarla a la batería de pruebas de regresión, pero no poseen conocimientos de Git, Python ni SQL. ¿Qué solución de Databricks resuelve este flujo?
  * *Respuesta:* La **Mosaic AI Review App**. Proporciona una interfaz web segura y sin código donde los expertos del dominio (SMEs) revisan las interacciones con feedback negativo, redactan la respuesta ideal (*Ground Truth*) y la publican directamente al **Golden Dataset** en Delta Lake para alimentar `mlflow.evaluate`.

* **6. Garantía de reproducibilidad en pipelines CI/CD:**
  * *Pregunta:* En un workflow de CI/CD que evalúa candidatos a producción contra el Golden Dataset con `mlflow.evaluate()`, ¿por qué es mandatorio fijar `temperature = 0.0` en el endpoint del modelo evaluador?
  * *Respuesta:* Para eliminar la varianza estocástica en el muestreo de tokens del juez. Fijar la temperatura en cero garantiza un comportamiento estrictamente determinista: ante las mismas entradas y rúbricas, el evaluador siempre emitirá exactamente el mismo puntaje numérico y justificación Chain-of-Thought, evitando falsos positivos en el gatekeeper.

---

## Conclusiones y Próximos Pasos

Evaluar aplicaciones de IA generativa no consiste en cruzar los dedos esperando que el modelo responda bien. Exige rigor científico: desglosar el sistema en etapas, auditar el recuperador con métricas de ranking, calibrar jueces LLM con rúbricas Chain-of-Thought y construir datasets dorados respaldados por el Lakehouse.

Con esto completamos la **Fase 4 (Evaluación y Gobernanza)** en su vertiente de calidad. En el siguiente módulo abordaremos el pilar complementario indispensable para la seguridad empresarial: **Módulo 6: Seguridad, Guardrails de Contenido y Prevención de Inyecciones en Mosaic AI**.
