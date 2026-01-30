---
title: "Capítulo 4: Limpieza de Datos - Traduciendo tus Primeros Pasos de Power Query a Pandas"
seoTitle: "Guía: Limpieza de Datos con Power Query vs. Python y Pandas"
seoDescription: "Aprende a traducir tus habilidades de limpieza de datos de Power Query a Python con Pandas. Una guía paso a paso para analistas de BI, usando datos reales."
datePublished: Thu Sep 11 2025 14:50:29 GMT+0000 (Coordinated Universal Time)
cuid: cmffizk2s000202l4cy568hw2
slug: capitulo-4-limpieza-de-datos-traduciendo-tus-primeros-pasos-de-power-query-a-pandas
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1757601856542/03502486-c71f-4a61-a466-477d12880f22.png
tags: tutorial, data, python, data-science, machine-learning, pandas, jupyter-notebook, espanol, data-cleaning, colab, power-bi, powerquery, honduras, bitoml

---

¡Bienvenido de nuevo a "El Puente al Machine Learning"!

En el capítulo anterior, conectamos nuestro entorno a un origen de datos. Ahora, vamos a arremangarnos y hacer lo que define el 80% del trabajo de un analista: **la limpieza de datos.**

Seguro te ha pasado. Abres un archivo CSV o de Excel y te encuentras con el caos. En Power Query, resolverlo es una secuencia de clics que ya dominamos. Pero, ¿cómo se ve esa batalla en Python? Hoy lo descubriremos.

### Nuestro Desafío: Un CSV "Sucio" del Mundo Real

Para este ejercicio, usaremos el archivo **elecciones_honduras_2017.csv** de nuestro repositorio. Este dataset, cuyos datos originales provienen de las fuentes oficiales del Tribunal Supremo Electoral (TSE), simula un reporte típico con algunos problemas comunes de formato, perfecto para practicar.

### El Mundo que Conocemos: Limpieza Paso a Paso en Power Query

En Power Query, el proceso es muy visual. Vamos a seguir la secuencia de clics que nos llevaría de un archivo "sucio" a una tabla lista para analizar.

**Paso 1: La Primera Mirada a los Datos Crudos**  
Al conectar nuestro CSV, nos encontramos con la vista previa inicial. Vemos filas en blanco en la parte superior e inferior, y los encabezados no están donde deberían.

<img src="https://raw.githubusercontent.com/NORSAB/El-Puente-al-Machine-Learning/main/imagenes/PBIX2.png" alt="Vista inicial de los datos en Power Query" style="max-width: 100%; height: auto;" />

**Paso 2: Eliminar Filas en Blanco**  
Nuestra primera acción es deshacernos de esas molestas filas completamente vacías. Para ello, vamos a la pestaña `Inicio` -> `Quitar filas` y seleccionamos `Quitar filas en blanco`.

<img src="https://raw.githubusercontent.com/NORSAB/El-Puente-al-Machine-Learning/main/imagenes/PBIX3.png" alt="Opción para quitar filas en blanco en Power Query" style="max-width: 100%; height: auto;" />

**Paso 3: Eliminar Filas Superiores**  
Ahora que las filas en blanco han desaparecido, vemos que todavía hay dos filas de texto irrelevante antes de nuestros verdaderos encabezados. Las eliminamos yendo a `Quitar filas` -> `Quitar filas superiores`.

<img src="https://raw.githubusercontent.com/NORSAB/El-Puente-al-Machine-Learning/main/imagenes/PBIX4.png" alt="Opción para quitar filas superiores en Power Query" style="max-width: 100%; height: auto;" />

En el cuadro de diálogo, especificamos que queremos eliminar las primeras **2** filas.

<img src="https://raw.githubusercontent.com/NORSAB/El-Puente-al-Machine-Learning/main/imagenes/PBIX5.png" alt="Especificando el número de filas a quitar" style="max-width: 100%; height: auto;" />

**Paso 4: Promover Encabezados**  
¡Casi lo tenemos! La primera fila de nuestros datos contiene ahora los nombres de las columnas. Usamos la mágica opción `Usar la primera fila como encabezado`.

<img src="https://raw.githubusercontent.com/NORSAB/El-Puente-al-Machine-Learning/main/imagenes/PBIX6.png" alt="Promover la primera fila como encabezado en Power Query" style="max-width: 100%; height: auto;" />

El resultado es una tabla mucho más limpia, con los encabezados en su lugar correcto.

<img src="https://raw.githubusercontent.com/NORSAB/El-Puente-al-Machine-Learning/main/imagenes/PBIX7.png" alt="Datos con encabezados promovidos correctamente" style="max-width: 100%; height: auto;" />

**Paso 5: Elegir las Columnas Relevantes**  
Finalmente, para nuestro análisis nos enfocaremos en las tres fuerzas políticas principales y otros datos clave. Hacemos clic en `Elegir columnas` en la pestaña de `Inicio`.

<img src="https://raw.githubusercontent.com/NORSAB/El-Puente-al-Machine-Learning/main/imagenes/PBIX8.png" alt="Seleccionando la opción de Elegir Columnas" style="max-width: 100%; height: auto;" />

En la ventana emergente, seleccionamos solo las columnas que necesitamos para nuestro análisis.

<img src="https://raw.githubusercontent.com/NORSAB/El-Puente-al-Machine-Learning/main/imagenes/PBIX9.png" alt="Ventana para seleccionar las columnas deseadas" style="max-width: 100%; height: auto;" />

¡Y listo! Ahora tenemos una tabla limpia y enfocada, lista para ser cargada al modelo. Al igual que en Python podemos tener múltiples DataFrames, en Power Query podríamos duplicar esta consulta para crear diferentes vistas si fuera necesario.

<img src="https://raw.githubusercontent.com/NORSAB/El-Puente-al-Machine-Learning/main/imagenes/PBIX10.png" alt="Tabla final limpia y lista en Power Query" style="max-width: 100%; height: auto;" />

Ahora, veamos la traducción de estos pasos a código.

---

## Cruzando el Puente: Limpieza con Pandas

Abre tu **Jupyter Lab** y crea un nuevo notebook en la carpeta `notebooks`.

### Paso 1: Cargar los Datos (La Forma Inteligente)
Si intentáramos leer el archivo directamente, Pandas se confundiría con las filas vacías y los encabezados desordenados. Pero, al igual que Power Query, podemos darle instrucciones precisas en una sola línea.

```python
import pandas as pd

# Definimos la ruta de nuestro archivo
ruta_elecciones = '../datos/elecciones_honduras_2017.csv'

# Leemos el archivo, pero esta vez hacemos varias cosas a la vez:
# 1) skiprows=11: Ignoramos las primeras 11 filas (ruido/espacios iniciales que Power Query quitó).
# 2) sep=';':      Delimitador típico en CSVs regionales (ajusta a ',' si tu archivo usa comas).
# 3) encoding:     Si ves caracteres raros en tildes/ñ, prueba 'utf-8' o 'latin-1'.
# 4) engine y bad lines: engine='python' es más robusto con CSVs “difíciles”; on_bad_lines='warn' ignora filas corruptas sin romper el flujo.
df_elecciones = pd.read_csv(
    ruta_elecciones,
    skiprows=11,
    sep=';',
    encoding='utf-8',
    engine='python',
    on_bad_lines='warn'
)

# Eliminamos filas que estén completamente en blanco (NaN)
df_elecciones.dropna(how='all', inplace=True)

# Visualizamos las primeras filas para confirmar que los datos se cargaron correctamente
df_elecciones.head()
```

**¿Por qué usamos cada función aquí?**  
- `import pandas as pd`: define el alias estándar para escribir funciones de Pandas de forma corta y legible.  
- `ruta_elecciones = ...`: centraliza la ubicación del archivo para cambiarla en un solo lugar si mueves el CSV.  
- `pd.read_csv(..., skiprows=11, sep=';', encoding='utf-8', engine='python', on_bad_lines='warn')`:  
  - `skiprows=11` salta el “ruido” inicial (texto/filas en blanco) para que la primera fila útil quede alineada con los encabezados.  
  - `sep=';'` fija el separador típico de CSVs regionales; si tu archivo usa coma, cámbialo a `sep=','`.  
  - `encoding='utf-8'` ayuda con tildes/ñ; si ves caracteres raros, intenta `latin-1`.  
  - `engine='python'` es más tolerante con CSVs complejos;  
  - `on_bad_lines='warn'` registra y salta líneas corruptas sin detener la ejecución.  
- `df_elecciones.dropna(how='all', inplace=True)`: elimina únicamente las filas totalmente vacías (todas sus celdas `NaN`), sin tocar registros con datos parciales.  
- `df_elecciones.head()`: verificación rápida de que columnas y filas se cargaron como esperabas (puedes complementar con `df_elecciones.info()` para tipos de datos).

---

### Paso 2: Seleccionar las Columnas que Necesitamos
Ahora, seleccionaremos las columnas relevantes para nuestro análisis.

```python
# Creamos una lista con los nombres de las columnas que queremos conservar
columnas_deseadas = [
    'DEPARTAMENTO',
    'MUNICIPIO',
    'PARTIDO ALIANZA PATRIOTICA HONDUREÑA',  # si el nombre exacto difiere, ajusta aquí
    'PARTIDO LIBERAL DE HONDURAS',
    'PARTIDO NACIONAL DE HONDURAS',
    'BLANCOS',
    'NULOS',
    'FECHA DE RECEPCION',
    'LATITUD',
    'LONGITUD'
]

# Creamos un nuevo DataFrame solo con estas columnas
df_limpio = df_elecciones[columnas_deseadas]

# Visualizamos el nuevo DataFrame limpio
df_limpio.head()
```

<img src="https://raw.githubusercontent.com/NORSAB/El-Puente-al-Machine-Learning/main/imagenes/Notebook2.png" alt="Salida de df.head() en el notebook local (ambiente Conda/Anaconda)" style="max-width: 100%; height: auto;" />

**¿Por qué usamos cada función aquí?**  
- Lista `columnas_deseadas`: documenta explícitamente el subconjunto que te interesa, mantiene el orden de columnas y es fácil de reutilizar.  
- `df_elecciones[columnas_deseadas]`: indexación por lista para filtrar columnas en un solo paso, creando una vista consistente para el análisis.  
- Asignación a `df_limpio`: separa el dataset “crudo” del dataset “listo”, evitando mezclar transformaciones y facilitando depuración.  
- `df_limpio.head()`: confirma que el filtrado se aplicó correctamente y que los nombres coinciden con lo esperado.

---

###  Notebook del capítulo en GitHub

Puedes **reutilizar el notebook existente** del repositorio *El Puente al Machine Learning* o, mejor aún, **crear el tuyo propio paso a paso** para comprender a fondo cada instrucción.

[![Abrir notebook en GitHub](https://img.shields.io/badge/Abrir%20notebook%20en%20GitHub-Click%20aqu%C3%AD-black?logo=github&logoColor=white)](https://github.com/NORSAB/El-Puente-al-Machine-Learning/blob/main/notebooks/Cap%C3%ADtulo%204%20Limpieza%20de%20Datos%20%20Traduciendo%20Tus%20Primeros%20Pasos%20de%20Power%20Query%20a%20Pandas.ipynb)

---

###  Exploración de alternativas: **Pandas vs Polars vs Dask** (¿y por qué no **PySpark** aquí?)

Como analistas de datos, lo natural tras dominar lo básico es **explorar alternativas** que optimicen *rendimiento, memoria y escalabilidad*. Preparé en **Google Colab** una comparativa práctica entre **Pandas**, **Polars** y **Dask**.  
**No incluí PySpark** por ahora: es especialista en *Big Data* (brilla con clústeres y archivos masivos), pero **en local (Conda/Anaconda) o Colab “vanilla”** su **arranque y puente Python↔JVM (Py4J)** añaden latencia y complejidad que no aportan valor para datasets pequeños/medianos de este capítulo.

**¿Por qué usé Google Colab?**  
- Evito **modificar tu ambiente** (Conda/Anaconda): instalo versiones específicas (`polars`, `dask`) sin romper dependencias locales.  
- Entorno **efímero y limpio**: ideal para pruebas y *benchmarks* rápidos.  
- En el futuro planeo **ajustar el entorno local** para ejecutar todo desde Conda (incluido Spark), pero hoy la ruta más estable es Colab.

**Comparativo rápido**

| Herramienta | Modelo | Ventajas | Limitaciones | Tamaño ideal de datos (aprox.) | Úsalo cuando… |
|---|---|---|---|---|---|
| **Pandas** | In-memory, single-thread | Madurez, API conocida, ecosistema gigante | Se degrada si no cabe en RAM | Hasta ~5–10 GB (según RAM) | Dataset cabe en memoria y priorizas rapidez de desarrollo |
| **Polars** | Columnar (Rust), *lazy* | Muy rápido, baja memoria, *lazy frames* | Menos extensiones que Pandas (crece rápido) | Hasta ~20–50 GB en una máquina potente | Buscas máximo rendimiento en una máquina |
| **Dask** | Paralelismo, *out-of-core* | Escala “tipo Pandas”, trabaja sobre datos que exceden RAM | Overhead de planificación/distribución | >10 GB, o particionado / múltiples archivos | Datos grandes o quieres paralelizar en tu equipo |

####  Notebook comparativo en Colab
[![Abrir en Google Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1w0bO_VsFVJpmaQQYeiISnX7PhDRASxoJ?usp=sharing)

> Podés **reutilizar mi notebook** o, mejor aún, **crear el tuyo propio paso a paso** para comprender a fondo las diferencias. A continuación, **copio exactamente los bloques de código** tal como están en el notebook de Colab, respetando los espacios.

---

### Paso 0: Preparación del Entorno
```python
!wget https://raw.githubusercontent.com/NORSAB/El-Puente-al-Machine-Learning/main/datos/elecciones_honduras_2017.csv
```

### Solución 1: Pandas (La Vía Conocida)
```python
import pandas as pd

ruta_elecciones = 'elecciones_honduras_2017.csv'

columnas_deseadas = [
    'DEPARTAMENTO',
    'MUNICIPIO',
    'PARTIDO ALIANZA PATRIOTICA HONDUREÑA',
    'PARTIDO LIBERAL DE HONDURAS',
    'PARTIDO NACIONAL DE HONDURAS',
    'BLANCOS',
    'NULOS',
    'FECHA DE RECEPCION',
    'LATITUD',
    'LONGITUD'
]

# Leemos el CSV con los parámetros necesarios
# engine='python' es más robusto para CSVs complejos
# on_bad_lines='warn' ignora filas corruptas sin romper el flujo
df_pandas = pd.read_csv(
    ruta_elecciones,
    skiprows=11,
    engine='python',
    sep=';',
    on_bad_lines='warn',
    encoding='utf-8'
)
df_pandas.dropna(how='all', inplace=True)

# Seleccionamos las columnas
df_pandas_limpio = df_pandas[columnas_deseadas]

print("Resultado con Pandas:")
display(df_pandas_limpio.head())
```

<img src="https://raw.githubusercontent.com/NORSAB/El-Puente-al-Machine-Learning/main/imagenes/Colab2.png" alt="Salida de Pandas en Colab" style="max-width: 100%; height: auto;" />

### Solución 2: Polars (El Retador Veloz)
```python
# En Colab, podemos instalar librerías directamente con !pip
!pip install polars

import polars as pl

# Polars tiene una sintaxis similar para leer CSVs
# Nota: si ves caracteres raros, usa encoding='latin1' o 'utf8-lossy' en pl.read_csv
df_polars = pl.read_csv(
    ruta_elecciones,
    skip_rows=11,
    separator=';',
    try_parse_dates=True
    # , encoding='utf8'  # por defecto; alternativas: 'utf8-lossy', 'latin1'
)

# En Polars, las transformaciones se encadenan de forma muy eficiente.
# Es el equivalente a nuestros dos pasos de Pandas en uno solo.
df_polars_limpio = df_polars.drop_nulls().select(columnas_deseadas)

print("Resultado con Polars:")
display(df_polars_limpio.head())
```

<img src="https://raw.githubusercontent.com/NORSAB/El-Puente-al-Machine-Learning/main/imagenes/Colab3.png" alt="Salida de Polars en Colab" style="max-width: 100%; height: auto;" />

### Solución 3: Dask (Para los Grandes Volúmenes)
```python
# Instalamos Dask y sus dependencias para el manejo de dataframes
# !pip install "dask[dataframe]"

import dask.dataframe as dd

# La lectura es casi idéntica a Pandas, pero Dask es "perezoso" (lazy)
# No carga los datos hasta que se lo pedimos explícitamente.
# Nota: si ves caracteres raros, agrega encoding='utf-8' o 'latin-1' a dd.read_csv
df_dask = dd.read_csv(
    ruta_elecciones,
    skiprows=11,
    sep=';',
    on_bad_lines='warn',
    assume_missing=True, # Ayuda a Dask a inferir tipos de datos con nulos
    blocksize=None # Leemos el archivo como una sola partición para este ejemplo
    # , encoding='utf-8'
)

# Las transformaciones también son perezosas
df_dask = df_dask.dropna(how='all')
df_dask_limpio = df_dask[columnas_deseadas]

# Le pedimos a Dask que "calcule" el resultado final con .compute()
resultado_dask = df_dask_limpio.compute()

print("Resultado con Dask:")
display(resultado_dask.head())
```

<img src="https://raw.githubusercontent.com/NORSAB/El-Puente-al-Machine-Learning/main/imagenes/Colab4.png" alt="Salida de Dask en Colab" style="max-width: 100%; height: auto;" />

---

## Conclusión: Clics vs. Código Reproducible

Hoy hemos visto cómo una secuencia de clics en Power Query se traduce a unas pocas líneas de código en Pandas. Ambos caminos nos llevan al mismo resultado: **una tabla limpia**. La gran ventaja del código es que esta "receta" de limpieza es **100% reproducible y automatizable**.

## Preparándonos para el Quinto Blog…

Ahora que nuestros datos tienen la forma correcta, el siguiente desafío es asegurar que tengan el contenido correcto. ¿Qué pasa si la columna de votos se leyó como texto? ¿O si hay valores nulos? En el próximo capítulo, nos sumergiremos en el mundo de **los tipos de datos y el manejo de valores nulos**.

**Ahora te toca a ti:** ¿Cuál es la tarea de limpieza de datos que más tiempo te consume en tu día a día?  
Si querés, **comparte un screenshot** de tu `df.head()` limpio **o** contá tu peor *pesadilla de datos sucios* en los comentarios.

#MachineLearning #BItoML #Pandas #PowerQuery #Python #DataScience #CienciaDeDatos #Español #Tutorial #DataCleaning #Honduras #Elecciones #Polars #Dask #DatosElectorales
