---
title: "Módulo 3: Desarrollo de Agentes de IA y Orquestación con Mosaic AI y MLflow"
seoTitle: "Desarrollo de Agentes de IA con Mosaic AI y MLflow	"
seoDescription: "Aprende a desarrollar, gobernar y depurar agentes de IA utilizando Mosaic AI Agent Framework, Unity Catalog y MLflow Tracing en Databricks.	"
datePublished: 2026-07-01T00:56:24.115Z
cuid: cmr1d7hut000009gf6i5s5nnp
slug: modulo-3-desarrollo-agentes-ia-mosaic-ai-mlflow
cover: https://cdn.hashnode.com/uploads/covers/689caf5773492fba4b653597/537be3dd-42a5-457e-bb43-8b2dfcb7a131.png
ogImage: https://cdn.hashnode.com/uploads/og-images/689caf5773492fba4b653597/d4c1b1e6-2cd5-4404-aba7-e3c2637b313e.png
tags: databricks, mlflow, ai-agents, llmops, mosaic-ai

---

Los chatbots tradicionales que solo responden preguntas basadas en un buscador de documentos ya no son suficientes para las demandas empresariales actuales. Las organizaciones necesitan sistemas capaces de tomar decisiones autónomas, interactuar con bases de datos transaccionales, actualizar registros de clientes y ejecutar flujos de trabajo complejos.

Aquí es donde entran los **Agentes de IA**. A diferencia de los pipelines lineales de RAG, los agentes tienen la capacidad de interactuar activamente con su entorno a través de herramientas (*tools*), evaluando los resultados de cada acción antes de entregar una respuesta final.

En esta guía te explicaré cómo diseñar, desarrollar y gobernar agentes inteligentes utilizando la infraestructura nativa de Databricks: **Mosaic AI Agent Framework**, **Unity Catalog** y **MLflow Tracing**. Todo lo que necesitas dominar para el examen de certificación **Databricks Certified Generative AI Engineer Associate**.

---

## 1. El salto de RAG simple a Agentes Autónomos

Para entender la arquitectura de un agente, primero debemos contrastarla con un sistema RAG tradicional:

*   **RAG Estático (Retrieval-Augmented Generation):** Sigue una ruta lineal predecible. La consulta del usuario se convierte en un vector, se buscan fragmentos similares en una base de datos vectorial, se inyectan en un prompt y el LLM genera una respuesta. Es un flujo pasivo de "solo lectura".
*   **Agente de IA (Bucle Dinámico):** El modelo de lenguaje no es solo el generador de texto; es el núcleo que toma las decisiones lógicas. Evalúa el objetivo del usuario, decide qué herramientas necesita, extrae los parámetros, ejecuta la acción, analiza el resultado y decide si ha cumplido la tarea o si requiere ejecutar más pasos.

El motor de un agente se basa en el bucle **Reasoning-Action (ReAct)**:

![El Bucle de Ejecución del Agente (ReAct Loop)](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/Modulo_3/Ciclo%20iterativo%20para%20razonar%2C%20ejecutar%20herramientas%20y%20responder.png)

Este comportamiento dinámico introduce un gran desafío de costos y control: si el agente se confunde o una herramienta devuelve un error, el sistema puede entrar en un bucle infinito de llamadas al LLM. Por lo tanto, en producción es obligatorio configurar un **límite máximo de iteraciones** (típicamente entre 5 y 10 pasos) para abortar la ejecución de forma segura.

---

## 2. Unity Catalog Functions: La base de herramientas seguras

Un agente sin herramientas es solo un modelo de lenguaje convencional. Las herramientas permiten al agente conectarse con el mundo exterior. En el ecosistema de Databricks, las **Funciones de Unity Catalog (UC Functions)** son el estándar de oro para crear y gestionar estas herramientas.

### ¿Por qué registrar herramientas en Unity Catalog?

1.  **Seguridad y Gobernanza (RBAC):** Unity Catalog controla quién puede ejecutar cada función. Si tu agente intenta invocar una herramienta para buscar salarios, pero el usuario que inició la sesión no tiene permisos de lectura sobre esa función en el catálogo, Unity Catalog bloqueará la ejecución inmediatamente.
2.  **Auto-Documentación:** Para que un LLM decida usar una herramienta, necesita saber qué hace y qué parámetros requiere. Unity Catalog lee las descripciones (`COMMENT`) y los tipos de datos de los parámetros directamente de la base de datos y se los entrega al modelo en formato JSON durante la inferencia.
3.  **Portabilidad y Reusabilidad:** En lugar de codificar funciones localmente en cada notebook o aplicación web, las registras una vez en tu catálogo corporativo. Cualquier agente desarrollado por diferentes equipos puede consumirlas de inmediato.

### Ejemplo práctico: Registro de herramientas

Puedes crear herramientas escribiendo funciones directamente en SQL o en Python nativo dentro de Databricks:

#### Registro de una herramienta SQL
```sql
CREATE OR REPLACE FUNCTION catalog_prod.ventas.obtener_descuento_cliente(
  cliente_id INT COMMENT 'ID único del cliente en la base de datos'
)
RETURNS DOUBLE
COMMENT 'Retorna el porcentaje de descuento asignado a un cliente según su historial'
RETURN SELECT porcentaje_descuento 
       FROM catalog_prod.ventas.descuentos_activos 
       WHERE id_cliente = cliente_id;
```

#### Registro de una herramienta Python
```python
# Definición de la función de Python en un notebook de Databricks
def verificar_inventario_producto(product_id: int) -> str:
    """
    Consulta el inventario actual de un producto específico en el almacén central.
    
    Parameters:
    product_id (int): El identificador único del artículo de inventario.
    
    Returns:
    str: Detalle de existencias del producto en formato de texto.
    """
    import pandas as pd
    df = spark.table("catalog_prod.inventario.existencias").filter(f"id_producto = {product_id}").toPandas()
    if df.empty:
        return f"El producto {product_id} no existe en el inventario."
    return f"Producto {product_id}: {df['cantidad'].values[0]} unidades disponibles."

# Registro de la función en Unity Catalog
spark.udf.register("catalog_prod.inventario.verificar_inventario_producto", verificar_inventario_producto)
```

> [!IMPORTANT]
> Para el examen, recuerda que las descripciones (`COMMENT` y docstrings) y las anotaciones de tipo (`type hints`) no son documentación decorativa. El LLM depende enteramente de ellas para entender la semántica de la herramienta y realizar la **llamada de funciones (Tool Calling)** de forma correcta.

![Flujo de Traducción Semántica de la Herramienta](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/Modulo_3/Flujo%20de%20Traducci%C3%B3n%20Sem%C3%A1ntica%20de%20la%20Herramienta.png)

---

## 3. Desarrollo de Agentes con Mosaic AI Agent Framework

**Mosaic AI Agent Framework** es la solución empresarial de Databricks diseñada para unificar el ciclo de vida de desarrollo, pruebas y despliegue de los agentes de IA.

### Arquitectura de Integración

El framework permite estructurar la lógica del agente enlazando el modelo de lenguaje con las funciones registradas de la siguiente manera:

![Arquitectura de Integración de Mosaic AI Agents](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/Modulo_3/figura_1_arquitectura_agente.png)

El flujo técnico funciona así:
1.  El usuario envía una consulta al agente.
2.  El agente consulta al LLM (utilizando endpoints de **Mosaic AI Model Serving**) pasando el historial de conversación y las firmas de las herramientas de Unity Catalog.
3.  El LLM detecta que necesita información externa y devuelve una solicitud de llamada a herramientas (*Tool Call*) con los parámetros extraídos del texto.
4.  El orquestador del agente intercepta esta llamada, ejecuta la función de Unity Catalog en un entorno seguro y le inyecta la salida del resultado al LLM.
5.  El LLM procesa el resultado y genera la respuesta definitiva para el usuario.

El framework es totalmente compatible con frameworks populares como **LangChain** y **LlamaIndex**, lo que te permite importar tus desarrollos previos y montarlos directamente sobre la infraestructura administrada de Databricks.

---

## 4. Inspección de la caja negra: MLflow Tracing

Depurar un agente de IA en desarrollo o monitorearlo en producción es sumamente complejo debido a la naturaleza dinámica de los bucles de ejecución. Si el bot responde incorrectamente a un cliente, ¿dónde estuvo la falla?
*   ¿El LLM extrajo mal el ID de cliente del texto original?
*   ¿La función de Unity Catalog tardó demasiado y causó un timeout?
*   ¿El buscador vectorial devolvió un fragmento erróneo como contexto?
*   ¿El LLM final alucinó a pesar de recibir los datos correctos?

Para responder a esto sin adivinar, Databricks integra **MLflow Tracing**, una herramienta de trazabilidad gráfica diseñada específicamente para visualizar la ejecución paso a paso de aplicaciones GenAI complejas.


![Estructura de una Traza en MLflow](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/Modulo_3/Estructura%20de%20una%20Traza%20en%20MLflow.png)

### ¿Qué información captura MLflow Tracing?

*   **Llamadas anidadas (Spans):** Cada paso lógico en la cadena se visualiza como un bloque de tiempo para identificar qué parte del flujo introduce la mayor latencia.
*   **Entradas y Salidas exactas:** Puedes ver el texto crudo recibido y devuelto por cada función o llamada de API intermedia.
*   **Consumo de Tokens:** Registra la cantidad exacta de tokens de entrada, salida y totales para calcular los costos financieros por interacción.
*   **Historial de Prompts:** Muestra el prompt exacto estructurado por el orquestador en cada paso intermedio.

Para habilitar la trazabilidad automatizada en un flujo de LangChain, solo necesitas agregar dos líneas de código en tu notebook:

```python
import mlflow

# Habilitar el registro y trazabilidad automática de LangChain en MLflow
mlflow.langchain.autolog(log_traces=True)
```

---

## 5. Empaquetado y Registro de Agentes en Unity Catalog

Una vez que el agente funciona localmente en tu notebook, debes empaquetarlo de forma reproducible para llevarlo al entorno de producción.

### Paso 1: Declarar Dependencias de Herramientas
Debes indicarle a MLflow qué herramientas de Unity Catalog requiere tu agente. Esto se hace para asegurar que, al desplegar el agente en un servidor de inferencia, el sistema mantenga los accesos adecuados:

```python
from mlflow.models import set_model_dependency

# Especificar las dependencias de funciones que el agente necesita consumir
set_model_dependency(
    model_uri="runs:/id_de_ejecución/agente_ventas",
    dependency={
        "type": "uc_function",
        "name": "catalog_prod.ventas.obtener_descuento_cliente"
    }
)
```

### Paso 2: Registrar el Modelo en Unity Catalog
Utilizamos la interfaz estándar de MLflow para registrar la lógica del agente. Esto congela la versión de las librerías utilizadas, las variables de entorno y los prompts base:

```python
import mlflow

# Guardar la estructura del agente en la ejecución actual de MLflow
with mlflow.start_run():
    mlflow.pyfunc.log_model(
        artifact_path="agente_ventas",
        python_model=MiAgenteDeVentas(),  # Instancia de tu clase del agente
        registered_model_name="catalog_prod.modelos_ia.agente_atencion_clientes"
    )
```

Al registrarlo en Unity Catalog, el agente pasa a estar completamente gobernado. Podrás gestionar sus versiones, promoverlo a diferentes ambientes (desarrollo, pruebas, producción) mediante alias de modelo y heredar todos los controles de acceso del catálogo corporativo.

---

## Tips definitivos para el Examen de Certificación

Si estás estudiando para obtener la credencial de **Databricks Certified Generative AI Engineer Associate**, grábate estos conceptos clave del Módulo 3:

*   **Integración de herramientas gobernadas:** Ante cualquier pregunta sobre cómo conectar un agente de forma segura a bases de datos transaccionales, APIs de la empresa o flujos internos respetando la seguridad, la respuesta correcta es **Unity Catalog Functions**.
*   **Depuración de flujos lógicos dinámicos:** Si te plantean un problema donde el agente tiene latencia alta o comportamiento inesperado y necesitas auditar qué paso intermedio está fallando, la herramienta correcta es **MLflow Tracing**.
*   **Definición semántica para Tool Calling:** Si el LLM no logra identificar la herramienta correcta o pasa parámetros en formatos equivocados, la solución técnica es **enriquecer los comentarios de la función (`COMMENT`) y las anotaciones de tipo (`type hints`)** en la firma de la función de Unity Catalog.
*   **Prevención de costos descontrolados:** Para evitar cobros excesivos por bucles infinitos causados por respuestas fallidas de las herramientas, debes configurar siempre un **límite máximo de iteraciones** en el loop del agente.

#DATABRICKS #AGENTES #MOSAICAI #MLFLOW #TRACING #UNITYCATALOG #TOOLS #LANGCHAIN #LLMOps
