---
title: "Módulo 2: Arquitectura RAG Fragmentación, Limpieza de Datos e Indexación en Delta Lake"
seoTitle: "Databricks GenAI: Ingesta y Arquitectura RAG"
seoDescription: "Aprende ingesta incremental con Auto Loader, fragmentación de datos, Delta Sync Index, Reranking y gobernanza para el examen de Databricks."
datePublished: 2026-06-24T04:45:35.815Z
cuid: cmqrlba1c00000bksa8h20eij
slug: m-dulo-2-arquitectura-rag-fragmentaci-n-limpieza-de-datos-e-indexaci-n-en-delta-lake
cover: https://cdn.hashnode.com/uploads/covers/689caf5773492fba4b653597/5cf44b39-f232-4504-be6a-b82e921c8d22.png
ogImage: https://cdn.hashnode.com/uploads/og-images/689caf5773492fba4b653597/1cabf209-a47a-4d25-a057-96a1f5e5fd45.png
tags: python, data-engineering, databricks, generative-ai, rag

---

Para construir aplicaciones de inteligencia artificial en una empresa, afinar prompts no es suficiente. Los modelos de lenguaje generales no saben nada sobre las bases de datos de nuestro negocio. Además, tienen límites de contexto muy estrictos. Intentar pasarles manuales completos o miles de registros en una sola consulta solo va a disparar los costos, ralentizar el sistema y causar errores de memoria.

Ahí es donde entra **RAG (Retrieval-Augmented Generation)**. La idea detrás de este patrón es muy simple: en lugar de obligar al LLM a memorizar toda la información, le permitimos consultar una base de datos para recuperar lo relevante antes de contestar. Es igual a hacer un examen a libro abierto con un asistente veloz que nos pasa las páginas necesarias en el segundo exacto.

Pero el truco de RAG está en la recuperación. Si el motor de búsqueda le pasa fragmentos llenos de ruido o textos sin sentido, el modelo va a inventar respuestas. Por eso, en esta guía para preparar el examen de **Databricks Generative AI Engineer Associate**, veremos cómo diseñar un pipeline de ingesta sólido utilizando Delta Lake y Unity Catalog, cuidando la fragmentación y el filtrado.

---

## 1. El flujo de limpieza de contenido

Meter información cruda a un motor de búsqueda vectorial es ir directo al fracaso. Los documentos internos suelen tener encabezados de página repetitivos, avisos legales larguísimos, marcado HTML residual y metadatos inútiles que confunden a los embeddings. 

El pipeline para limpiar y preparar los textos consta de cinco pasos:

![Pipeline de Limpieza de Contenido Dark](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/Modulo_2/Pipeline%20de%20Limpieza%20de%20Contenido%20Dark.png)

* **Extraer los archivos crudos (Raw):** Primero, leemos documentos (PDFs, HTML o texto) desde buckets en la nube. Para automatizar esto de forma incremental y eficiente, lo ideal es usar **Databricks Auto Loader**, que detecta nuevos archivos conforme van llegando a la ruta.
* **Limpiar el ruido:** Mediante funciones nativas de Spark y expresiones regulares, eliminamos código de marcado, disclaimers legales repetitivos (boilerplate) y números de página. Si no quitamos este contenido vacío, el buscador vectorial empezará a recuperar estos textos genéricos para cualquier consulta.
* **Fragmentación inteligente:** El texto limpio se corta en trozos (*chunks*). Ajustar el tamaño y el solapamiento (*overlap*) de estos bloques depende del límite del LLM que vayamos a usar.
* **Vectorizar (Embeddings):** Pasamos cada fragmento por un modelo de embeddings usando endpoints de **Databricks Model Serving** para generar los vectores.
* **Escribir en Delta Lake:** Guardamos los fragmentos de texto limpitos, sus embeddings y metadatos clave (como la URI del archivo origen) en una tabla Delta de nivel Gold regulada por **Unity Catalog**.

---

## 2. Estrategias de fragmentación (Chunking)

El *chunking* es el arte de dividir textos largos en fragmentos que el modelo pueda digerir. Esta decisión define directamente el balance entre velocidad de búsqueda y la retención del contexto original.

El examen de Databricks evalúa nuestra capacidad para elegir el método correcto según cómo estén organizados los datos:

![Estrategias de Fragmentación](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/Modulo_2/figura_1_estrategias_chunking.png)

### A. Fragmentación de Longitud Fija (Fixed-Length)
Se corta el texto a partir de un conteo rígido de caracteres o tokens (por ejemplo, bloques exactos de 200 tokens).
* **Cuándo usarla:** Para logs de sistemas, archivos CSV estructurados o código de programación.
* **Ventajas:** Procesamiento inmediato y desarrollo muy simple.
* **Desventajas:** Rompe las oraciones de forma arbitraria. Puede cortar palabras clave por la mitad (como pasar de `"desarrollador"` a `"des ➔ arrollador"`), arruinando el significado en los límites del bloque.

### B. Fragmentación a Nivel de Oración (Sentence-Level)
Usa herramientas de procesamiento de lenguaje natural (como spaCy o NLTK) para encontrar los puntos finales y cortar solo donde termina una frase.
* **Cuándo usarla:** Glosarios, bases de preguntas frecuentes (FAQs) o textos muy directos.
* **Ventajas:** La sintaxis se mantiene limpia y las oraciones no quedan truncadas.
* **Desventajas:** Al aislar oraciones individuales, a veces se pierde el hilo conductor del párrafo si hay referencias cruzadas ("él", "esto").

### C. Fragmentación Basada en Párrafos (Paragraph-Based)
Usa los saltos de línea dobles (`\n\n`) como el punto de división natural.
* **Cuándo usarla:** Contratos, informes financieros de texto largo o artículos estructurados.
* **Ventajas:** Mantiene el contexto original que el autor organizó al escribir.
* **Desventajas:** La extensión de los párrafos humanos varía demasiado. Nos arriesgamos a tener fragmentos gigantescos e inconsistentes.

### D. Fragmentación por Ventana Deslizante (Sliding Window)
Crea fragmentos de tamaño fijo pero repitiendo una porción de tokens del bloque anterior en el nuevo (solapamiento o *overlap*). Por ejemplo, un bloque de 300 tokens que arrastra 50 tokens del fragmento previo.
* **Cuándo usarla:** Manuales técnicos complejos, documentos legales con cláusulas extensas o libros.
* **Ventajas:** Excelente para no perder la continuidad de las ideas en los puntos de corte.
* **Desventajas:** Al duplicar información, incrementa el tamaño de la base de datos vectorial y, por ende, los costos.

### E. Fragmentación Semántica (Semantic Chunking)
El enfoque más sofisticado. Calcula la similitud vectorial entre oraciones consecutivas. En el momento en que la instrución o similitud cae por debajo de un umbral que definamos, el sistema asume un cambio de tema y genera la división.
* **Cuándo usarla:** Actas de reuniones, transcripciones de audio o textos continuos sin subtítulos.
* **Ventajas:** Los cortes son completamente adaptativos e inteligentes.
* **Desventajas:** Requiere muchísima potencia de cómputo, ya que hay que generar embeddings de prueba para cada oración antes de hacer los cortes definitivos.

### Matriz de decisiones de fragmentación

Para que te quede más claro cómo elegir en el mundo real, organigrama de opciones según su **Complejidad de Implementación** y su **Criterio de Corte**:

![Matriz de Fragmentación RAG Dark](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/Modulo_2/Matriz%20de%20Fragmentaci%C3%B3n%20RAG%20Dark.png)

---

## 3. Control de granularidad y solapamiento

Ajustar el tamaño de los fragmentos y su solapamiento es un juego de equilibrios en RAG:

### El tamaño del fragmento (Granularidad)
* **Chunks Pequeños (100 - 200 tokens):** Encuentran datos específicos muy rápido y con alta precisión. ¿El problema? Tienen poca cobertura de contexto y nos obligan a gestionar millones de vectores, lo que puede ralentizar la consulta.
* **Chunks Grandes (500 - 800 tokens):** Le dan al LLM información de fondo muy rica para que redacte su respuesta. La desventaja es que los temas clave se diluyen, bajando la precisión de la búsqueda vectorial pura.

### El solapamiento (Overlap)
* **Overlap Fijo:** Repite un porcentaje estático (típicamente entre 10% y 20% del bloque). Es la opción más popular porque funciona sin complicaciones.
* **Overlap Dinámico:** El algoritmo es inteligente; si el corte cae a la mitad de una oración, extiende el bloque automáticamente hasta encontrar un punto o un fin de párrafo.

### El reto del parseo consciente del diseño (Layout-Aware Parsing)
Los extractores de texto básicos leen de izquierda a derecha. Si un PDF tiene dos columnas de texto, un parseador simple mezclará las líneas de ambas columnas, haciendo que el fragmento final sea ilegible.
Para solucionar esto, la certificación evalúa el uso de herramientas capaces de identificar la estructura visual (títulos, tablas, imágenes y columnas). Al procesar tablas, por ejemplo, el sistema debe convertirlas a formatos estructurados como HTML o Markdown dentro del chunk de texto, permitiendo que el modelo de embeddings entienda las relaciones de las celdas y filas.

---

## 4. Arquitectura Medallion y Gobernanza en Delta Lake

El almacenamiento de datos en la plataforma de Databricks sigue la estructura Medallion. Esta separación lógica nos ayuda a depurar y controlar la información que entra a nuestro buscador vectorial:

![Pipeline de Ingesta Delta](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/Modulo_2/figura_2_pipeline_ingesta_delta.png)

1. **Tabla Bronze (Crudo/Raw):** Ingesta inicial de los archivos mediante **Auto Loader** (usando `format("cloudFiles")` y `cloudFiles.format("binaryFile")`). Aquí guardamos los archivos PDF o binarios tal como vienen de la nube, con su nombre, tamaño y fecha.
2. **Tabla Silver (Limpio):** Almacena el texto extraído y normalizado. Se aplican herramientas de OCR (como Tesseract en MLflow) o librerías de parseo (como PyMuPDF) para eliminar el código HTML y las marcas vacías.
3. **Tabla Gold (Indexado RAG):** Contiene los fragmentos finales divididos en chunks, sus embeddings vectoriales y todos los metadatos de búsqueda estructurados (URI original, autor, permisos).

### Ingesta incremental con Spark Structured Streaming
En producción, el pipeline RAG se ejecuta como un flujo de transmisión continua (Structured Streaming) para procesar nuevos documentos en tiempo real. 
En lugar de mantener clústeres encendidos las 24 horas, Databricks permite configurar el streaming con el disparador **`.trigger(availableNow=True)`**. Esto procesa únicamente los nuevos archivos que han llegado desde la última ejecución en forma de micro-lotes y apaga el clúster automáticamente al terminar, optimizando drásticamente los costos.

### ¿Por qué Delta Lake es crítico en RAG?
* **predicate Pushdown (Filtros rápidos):** Permite filtrar los metadatos de la tabla Gold de forma veloz antes de hacer la búsqueda por vectores. Si buscas solo manuales del año 2026, Spark descarta el resto en la tabla Delta antes de comparar embeddings.
* **Time Travel (Auditoría):** Guarda el historial completo de cambios de la tabla. Si tu modelo empieza a alucinar hoy, puedes usar Time Travel para ver exactamente qué archivos estaban indexados ayer a las 3:00 PM y comparar versiones.
* **Actualizaciones transaccionales (ACID):** Permite hacer un `DELETE` de los chunks de un manual viejo o un `UPDATE` de un fragmento modificado de forma segura, sin tener que borrar e indexar de nuevo toda tu base de datos.

### Seguridad nativa con Unity Catalog
Unity Catalog hereda los permisos de acceso de los archivos originales. Al sincronizar la tabla Gold con el índice de Vector Search, la búsqueda respeta de forma automática el control de accesos basado en roles (RBAC). Si un empleado de ventas hace una pregunta al bot, el buscador vectorial filtrará los resultados para mostrarle únicamente fragmentos de archivos a los que tiene permiso de lectura en Unity Catalog.

---

## 5. El motor de búsqueda: Vector Search y Reranking

### Métricas de Similitud Vectorial
En el examen te evaluarán cuándo utilizar cada métrica en Databricks Vector Search:
* **Distancia Coseno (Cosine Distance):** Mide el ángulo entre dos vectores. Ignora el tamaño del documento, enfocándose solo en la dirección del significado. Es la métrica por defecto y más recomendada para RAG.
* **Producto Punto (Dot Product):** Mide la dirección y la longitud. Es la más rápida de calcular, pero **exige** que los embeddings estén normalizados (longitud igual a 1). Si usas modelos como OpenAI u Cohere, es la opción ideal.
* **Distancia Euclidiana (L2):** Mide la distancia en línea recta entre dos puntos. Se usa poco en texto, ya que penaliza fuertemente a los documentos largos.

### Sincronización del Índice de Vector Search
En Databricks, puedes crear dos tipos de índices vectoriales según la arquitectura del sistema:
* **Delta Sync Index (Administrado):** Se conecta a tu tabla Delta Gold en Unity Catalog. Databricks permite configurar este índice en dos modalidades de embeddings:
  1. **Managed Embeddings (Administrados):** Databricks calcula los embeddings de forma automática llamando a un endpoint de Model Serving (como BGE) cada vez que ingresa texto a la tabla Delta Gold.
  2. **Self-Managed Embeddings (Autoadministrados):** Tú calculas los vectores en tu pipeline de Spark y los guardas en una columna de la tabla Delta. El índice solo se encarga de sincronizarlos de forma incremental.
* **Direct Vector Access Index (No administrado):** No está enlazado a una tabla Delta. El desarrollador debe calcular los embeddings manualmente en su código y subirlos al índice mediante la API.

![Modelos de Generación de Embeddings Dark](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/Modulo_2/Modelos%20de%20Generaci%C3%B3n%20de%20Embeddings%20Dark.png)

### Manejo de borrados en la indexación (Hard vs. Soft Deletes)
Es crucial saber cómo se propagan las eliminaciones de documentos al índice vectorial:
* **Hard Delete:** Si eliminas físicamente una fila de la tabla Delta Gold (`DELETE FROM`), la sincronización de Vector Search eliminará de inmediato ese vector del índice.
* **Soft Delete:** Si marcas un documento como inactivo (ej. `activo = false`), el vector permanecerá en el índice. En este caso, debes filtrar explícitamente usando la columna de estado en la consulta del buscador vectorial.

### Reranking: Resolviendo el dilema de Precision vs. Recall
La búsqueda vectorial es excelente encontrando información relevante rápidamente (alto **Recall**), pero suele mezclar ruido irrelevante (baja **Precision**). Pasar 100 fragmentos crudos al prompt satura la ventana de contexto y confunde al LLM.

Para solucionar esto, aplicamos un embudo en dos fases:

![Mejora de la Búsqueda Vectorial con Reranking Dark](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/Modulo_2/Mejora%20de%20la%20B%C3%BAsqueda%20Vectorial%20con%20Reranking%20Dark.png)

1. **Primera etapa (Búsqueda Vectorial Rápida - Top-100):** Filtramos rápidamente la base de datos usando similitud de coseno para recuperar 100 candidatos. Es un paso veloz pero con ruido.
2. **Segunda etapa (Re-ranking Semántico - Top-5):** Pasamos los 100 candidatos por un modelo **Cross-Encoder** (Reranker). Este evalúa la pregunta del usuario y el fragmento juntos en una red neuronal, calculando una puntuación semántica exacta. Seleccionamos únicamente los 5 fragmentos con mayor puntuación para construir el prompt final.

Este embudo optimiza el prompt, ahorra costos de tokens y reduce drásticamente las alucinaciones del modelo:

![Precision y Recall RAG Funnel](https://raw.githubusercontent.com/NORSAB/Generative-AI-Engineer/main/Blog/figuras/Modulo_2/figura_3_precision_recall_rag.png)

---

## 6. Pipeline práctico en PySpark (Structured Streaming)

Este script de producción en Databricks implementa el pipeline de ingesta incremental utilizando Auto Loader y Structured Streaming. Lee nuevos documentos en tiempo real, los limpia, genera fragmentos por ventana deslizante y escribe los resultados en Delta de forma incremental con el disparador `availableNow`:

```python
import re
from pyspark.sql import SparkSession
from pyspark.sql.functions import current_timestamp, lit, udf
from pyspark.sql.types import StringType, ArrayType

# 1. Limpieza de ruido y marcado residual
def clean_document_text(raw_text):
    if not raw_text:
        return ""
    # Eliminar etiquetas HTML/XML
    text = re.sub(r'<[^>]+>', '', raw_text)
    # Quitar marcas de confidencialidad y disclaimers repetitivos
    text = re.sub(r'Confidencial - Propiedad de la Empresa.*', '', text, flags=re.IGNORECASE)
    # Normalizar espacios múltiples en blanco
    text = re.sub(r'\s+', ' ', text).strip()
    return text

# 2. Generador de fragmentos por ventana deslizante
def make_sliding_chunks(text, chunk_size=200, overlap=40):
    if not text:
        return []
    words = text.split(" ")
    chunks = []
    stride = chunk_size - overlap
    
    for i in range(0, len(words), stride):
        chunk_words = words[i:i + chunk_size]
        if chunk_words:
            chunks.append(" ".join(chunk_words))
    return chunks

# Registrar funciones UDF para procesamiento en Spark
clean_text_udf = udf(clean_document_text, StringType())
chunk_text_udf = udf(lambda t: make_sliding_chunks(t, 100, 20), ArrayType(StringType()))

# 3. Inicializar sesión de Spark
spark = SparkSession.builder.getOrCreate()

# Ruta de entrada de documentos crudos en Cloud Storage (Bronze)
source_directory = "/mnt/documentos/incoming_raw/"
checkpoint_directory = "/mnt/documentos/checkpoints/ingesta_rag"

# Lectura incremental usando Auto Loader
raw_stream_df = spark.readStream \
    .format("cloudFiles") \
    .option("cloudFiles.format", "text") \
    .load(source_directory)

# Aplicar pipeline de limpieza y chunking
processed_df = raw_stream_df \
    .withColumn("clean_content", clean_text_udf("value")) \
    .withColumn("chunks", chunk_text_udf("clean_content")) \
    .selectExpr("input_file_name() as source_uri", "explode(chunks) as content") \
    .withColumn("updated_at", current_timestamp())

# Escritura incremental segura con trigger AvailableNow (Gold)
query = processed_df.writeStream \
    .format("delta") \
    .outputMode("append") \
    .option("checkpointLocation", checkpoint_directory) \
    .trigger(availableNow=True) \
    .toTable("catalog_name.schema_name.rag_gold_chunks")

query.awaitTermination()
print("¡Pipeline Structured Streaming completado e indexado de forma incremental!")
```

Este flujo automatizado garantiza que cada fragmento guardado esté libre de ruido y estructurado con sus respectivos metadatos, estableciendo la base para búsquedas vectoriales rápidas y precisas en cualquier arquitectura RAG corporativa.
