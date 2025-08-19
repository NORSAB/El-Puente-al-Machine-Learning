---
title: "¡Manos a la Obra! Instalando tu Kit de Herramientas de Machine Learning"
seoTitle: "Guía 2025: Instalar Anaconda y Google Colab para Machine Learning"
seoDescription: "¿Vienes del mundo de BI y quieres empezar en Machine Learning? Aprende a instalar tu kit de herramientas desde cero con esta guía completa y paso a paso."
datePublished: Tue Aug 19 2025 02:18:26 GMT+0000 (Coordinated Universal Time)
cuid: cmehwzt01000a02jvdasy2m0u
slug: manos-a-la-obra-instalando-tu-kit-de-herramientas-de-machine-learning
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1755569868206/ab2b9c30-b7bc-4684-92ba-bc454b851345.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1755544162731/2c25eac7-d7fa-4e4a-b726-d34604283f60.png
tags: tutorial, python, data-science, machine-learning, pandas, jupyter, jupyter-notebook, espanol, anaconda, google-colab, bitoml, cienciadedatos

---

### ¡Manos a la Obra! Instalando tu Kit de Herramientas de Machine Learning

¡Bienvenido al segundo capítulo de "El Puente al Machine Learning"!

En nuestro primer post, compartí mi historia y la motivación que, como especialista en BI, me impulsó a cruzar hacia el fascinante mundo del Machine Learning. Ahora que tenemos claro el "porqué", es momento de construir nuestro taller.

Hoy vamos a poner manos a la obra. Te guiaré paso a paso para instalar y configurar tu primer kit de herramientas de ciencia de datos. Exploraremos los dos caminos más populares: la instalación local con **Anaconda** y la alternativa en la nube con **Google Colab**.

<!--  -->

---

### El Camino Local: Guía Práctica para Instalar Anaconda

Tener un entorno en tu propia máquina te da control total y es ideal para el trabajo del día a día. Anaconda es la navaja suiza para esto, ya que no solo instala Python, sino que gestiona todos nuestros paquetes y entornos de forma ordenada.

#### **Paso 1: Descargar Anaconda**

Lo primero es ir a la [página oficial de Anaconda](https://www.anaconda.com/download). Verás dos opciones principales: **Anaconda Distribution** y **Miniconda**.

Para esta guía, y como recomendación para empezar, descargaremos la **Anaconda Distribution** completa. Esta versión ya incluye cientos de las librerías más populares para ciencia de datos, lo que nos facilita mucho el arranque.

Elige el instalador para tu sistema operativo (Windows, macOS o Linux) y la versión más reciente de Python (ej. **Python 3.13** o superior).

> **Nota importante para usuarios de Mac Intel (2025):** A partir del 15 de agosto de 2025, Anaconda ha dejado de construir nuevos paquetes para Macs con procesadores Intel (arquitectura osx-64). Los instaladores actuales seguirán funcionando, pero para tener acceso a las últimas versiones de las librerías, se recomienda usar Google Colab o considerar hardware más reciente con Apple Silicon.

<img src="https://raw.githubusercontent.com/NORSAB/El-Puente-al-Machine-Learning/main/imagenes/Anaconda2.png" alt="Opciones de descarga de Anaconda Distribution y Miniconda" style="max-width: 100%; height: auto;">

---

#### **Paso 2: Instalar Anaconda**

Ejecuta el instalador. La mayoría de las veces, puedes seguir las recomendaciones por defecto. Haz clic en "Next" y acepta los términos.

> **Tip para usuarios de Windows:** Durante la instalación, verás una opción para "Add Anaconda to my PATH environment variable". Aunque la recomendación oficial es no marcarla para evitar conflictos, si eres principiante y solo usarás Anaconda para Python, marcarla puede facilitar el uso de comandos desde cualquier terminal. Si decides no hacerlo, ¡recuerda usar siempre la **Anaconda Prompt**!

<!--  -->

---

#### **Paso 3: Crear un Entorno Virtual Limpio (¡La Mejor Práctica!)**

Una vez instalado, no trabajaremos en el entorno "base". Crearemos un espacio aislado y limpio solo para nuestros proyectos de Machine Learning.

Abre la **Anaconda Prompt** (búscala en el menú de inicio) y escribe el siguiente comando:

```bash
conda create -n mi_entorno_ml python=3.13
```

* `conda create -n`: Es la instrucción para crear un nuevo entorno.
* `mi_entorno_ml`: Es el nombre que le damos. ¡Puedes ponerle el que quieras!
* `python=3.13`: Especificamos la versión de Python que queremos usar.

---

#### **Paso 4: Activar Nuestro Entorno**

Para empezar a usar nuestro nuevo entorno, debemos "activarlo". En la misma terminal, escribe:

```bash
conda activate mi_entorno_ml
```

Notarás que el texto al inicio de la línea en tu terminal cambiará de `(base)` a `(mi_entorno_ml)`. ¡Eso significa que ya estás dentro!

<!--  -->

---

#### **Paso 5: Instalar las Librerías Esenciales**

Ahora que estamos en nuestro entorno limpio, instalemos el kit básico que mencionamos en el post anterior. Escribe este comando:

```bash
conda install numpy pandas scikit-learn matplotlib
```

Anaconda se encargará de descargar e instalar estas librerías y todas sus dependencias.

> **Pro Tip:** Si alguna vez tienes problemas con los comandos, es una buena práctica asegurarte de que conda esté actualizado. Puedes hacerlo ejecutando `conda update conda`.

---

#### **Paso 6: Lanzando Nuestro Laboratorio (Anaconda Navigator)**

¡Ya casi terminamos! Ahora vamos a la parte visual. Busca y abre **Anaconda Navigator**. Esta es la interfaz gráfica desde donde puedes lanzar todas tus aplicaciones.

Asegúrate de que en el menú desplegable de arriba esté seleccionado tu entorno: **"mi_entorno_ml"**.

<img src="https://raw.githubusercontent.com/NORSAB/El-Puente-al-Machine-Learning/main/imagenes/Anaconda3.png" alt="Interfaz de Anaconda Navigator con varias aplicaciones para instalar" style="max-width: 100%; height: auto;">

Para empezar a trabajar con los notebooks que usaremos en los próximos tutoriales, necesitas instalar dos herramientas clave desde aquí:

1.  **Jupyter Notebook:** Es la herramienta principal para crear y compartir documentos que contienen código, visualizaciones y texto. Haz clic en el botón **"Install"** debajo de su ícono.
2.  **JupyterLab:** Es la versión más moderna y potente de Jupyter Notebook, con una interfaz más completa. También te recomiendo instalarla.

Una vez instalados, verás que el botón cambia a **"Launch"**. ¡Con eso, tu taller de Machine Learning está oficialmente abierto!

---

### La Alternativa en la Nube: Un Vistazo a Google Colab

Si prefieres no instalar nada, Google Colab es tu mejor amigo. Es un Jupyter Notebook que corre en los servidores de Google y al que accedes desde tu navegador.

**Ventajas:**

* **Cero instalación.**
* **Acceso** a GPUs **gratuitas**, que son esenciales para entrenar modelos complejos más adelante.
* **Fácil de compartir y colaborar**, como un Google Doc.

Simplemente ve a [colab.research.google.com](https://colab.research.google.com), inicia sesión con tu cuenta de Google y crea un nuevo notebook. ¡Ya estás listo para escribir código!

> **Novedad 2025:** Colab ahora integra funciones de IA, como la ayuda de Gemini, para autocompletar y explicar código. ¡Pruébalo para acelerar tu aprendizaje!

```python
print("¡Hola desde Colab! Mi laboratorio en la nube está listo. ☁️")
```

---

### Conclusión: ¿Cuál Camino Tomar?

Ambas opciones son fantásticas, pero sirven para propósitos ligeramente distintos.

| Característica | Anaconda (Local) | Google Colab (Nube) |
| :--- | :--- | :--- |
| **Instalación** | Requerida en tu PC | Ninguna |
| **Rendimiento** | Depende de tu hardware (RAM, CPU) | Usa los servidores de Google |
| **Conexión** | Funciona 100% offline | Requiere conexión a internet |
| **Ideal para...** | Proyectos serios, control total | Empezar rápido, colaborar, usar GPUs |

Mi recomendación: **¡Usa ambos!** Empieza con Colab para experimentar rápidamente y usa Anaconda para tus proyectos más establecidos.

---

### Preparándonos para el Tercer Blog...

¡Felicidades! Ya sea en la nube o en tu máquina, tu laboratorio de ciencia de datos está listo. Ahora que tenemos las herramientas, en nuestro próximo capítulo vamos a empezar a usarlas de verdad.

Y aquí es donde conectaremos directamente con lo que ya dominamos. Si vienes del mundo de BI, seguramente has pasado horas usando **SQL** para extraer datos y **Power Query** para limpiarlos y transformarlos. En nuestro próximo post, nos enfocaremos en este último: te mostraré cómo podemos replicar las tareas de limpieza que hacemos en Power Query, pero esta vez con el poder de **Python y la librería Pandas**.

Dejaremos la comparación con SQL para un futuro capítulo. Por ahora, ¡prepárate para traducir lo que ya sabes de Power Query a un nuevo y poderoso lenguaje de manipulación de datos! ¡Nos vemos en la próxima entrega para seguir cruzando juntos este puente! 🚀

**¡Ahora te toca a ti! ¿Cuál camino elegiste: Anaconda o Colab? ¡Cuéntame en los comentarios si tuviste algún tropiezo y cómo lo resolviste!**

`#MachineLearning #BItoML #Anaconda #GoogleColab #Python #DataScience #CienciaDeDatos #Español #Tutorial #Jupyter #Pandas`
