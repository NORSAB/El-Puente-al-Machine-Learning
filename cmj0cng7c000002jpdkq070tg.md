---
title: "El Especialista en Tendencias (Prophet de Meta)"
seoTitle: "Pronóstico de Aprobaciones BCIE con Prophet Machine Learning con Datos"
seoDescription: "Aprende a utilizar Prophet de Meta para predecir tendencias financieras. Un caso de estudio real usando Python y datos abiertos del BCIE para proyectar"
datePublished: Wed Dec 10 2025 18:35:23 GMT+0000 (Coordinated Universal Time)
cuid: cmj0cng7c000002jpdkq070tg
slug: pronostico-aprobaciones-bcie-prophet-tutorial
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1765391369225/83ca7322-4a7f-45c8-a5c0-5099b87fac76.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1765391557317/8bfe905d-b2bb-458b-bf4d-1faa60440133.png
tags: data-science, machine-learning, data-analysis, time-series, finance, open-data, bcie

---

Bienvenidos de nuevo a **"El Puente al Machine Learning"**.

En el [Capítulo 1](https://normansabillon.hashnode.dev/explorando-el-desa), hicimos algo fundamental: conectamos los cables. Descargamos 609 registros históricos del **Portal de Datos Abiertos del BCIE**, los limpiamos y descubrimos una verdad contraintuitiva: **el banco está aprobando menos proyectos en cantidad, pero con montos mucho mayores**.

Hoy, dejamos de mirar el espejo retrovisor. Hoy miramos hacia adelante.

Vamos a utilizar **Prophet**, el algoritmo de código abierto desarrollado por el equipo de Core Data Science de Meta (Facebook), para intentar responder la pregunta del millón (literalmente): **¿Cuánto dinero aprobará el BCIE en 2026, 2027 y hasta 2030?**

> *Todo el código de este capítulo, incluyendo los scripts para generar las proyecciones y gráficos, está disponible en el repositorio oficial del laboratorio: [models/aprobaciones_prophet](https://github.com/NORSAB/Datos-Abiertos-BCIE/tree/e624a0a9c92736d1d2a81a551e92597097753a92/models/aprobaciones_prophet).*

---

<a name="resumen-rapido"></a>
### Resumen Rápido

* **Qué:** Construimos el primer modelo predictivo de la serie para proyectar las aprobaciones financieras del BCIE (2026-2030).
* **Herramienta:** **Prophet de Meta**, ideal para series temporales con tendencias fuertes y eventos estacionales.
* **Hallazgo:** El modelo proyecta una tendencia de **crecimiento sostenido** para el banco en el próximo quinquenio, superando barreras históricas de aprobación total hacia el 2029.
* **Código:** Implementamos un bucle iterativo que entrena un modelo independiente para cada combinación de País/Sector.

---

<a name="recapitulando"></a>
### Recapitulando: El hallazgo del Capítulo 1

Antes de predecir, recordemos qué estamos modelando. En el capítulo anterior, el dashboard de Power BI nos reveló que la estrategia del banco ha cambiado. Ya no vemos decenas de préstamos pequeños. Ahora vemos "Megaproyectos".

Esto es crucial para nuestro modelo: no estamos prediciendo *cuántos* préstamos se aprobarán (un problema de conteo), sino el **monto total en dólares** (un problema de regresión de series temporales).

---

<a name="por-que-prophet"></a>
### ¿Por qué Prophet? El "Oráculo" de Meta

Para nuestros lectores que vienen de **Finanzas o Economía**: imaginad un analista junior incansable que no se asusta si faltan datos de un año o si hay un pico repentino por una pandemia. Eso es Prophet.

A diferencia de modelos clásicos (como ARIMA) que requieren un ajuste manual meticuloso de parámetros (p, d, q), Prophet brilla en dos cosas que necesitamos desesperadamente para los datos del BCIE:

1.  **Resiliencia a "Shocks":** El BCIE no es un reloj suizo; es un organismo de desarrollo. Tiene picos por emergencias (COVID-19, Huracanes Eta/Iota) y valles por cambios estratégicos. Prophet maneja estos *outliers* sin romper la tendencia general.
2.  **Interpretación Humana:** Prophet no es una "caja negra". Su fórmula es aditiva, muy parecida a como pensamos nosotros:

```text
y(t) = Tendencia + Estacionalidad + Eventos + Error
```

Para los **Data Scientists**: Usamos Prophet porque automatizar el ajuste de hiperparámetros para más de 20 series temporales distintas con ARIMA sería una pesadilla de mantenimiento. Prophet nos da un *baseline* robusto "out-of-the-box" que maneja muy bien los datos faltantes (missing data), algo común en registros históricos de desarrollo.

---

<a name="la-estrategia"></a>
### La Estrategia: Divide y Vencerás

Aquí está el secreto de nuestro enfoque técnico. No lanzamos todos los datos a una licuadora gigante para predecir un solo número total del banco. **Eso sería un error gravísimo.**

El comportamiento del **Sector Público** (proyectos de infraestructura masiva y largo plazo) no tiene nada que ver con el **Sector Privado** (dinámicas de mercado más rápidas y volátiles).

Por eso, nuestra estrategia en el notebook es granular:
1.  Agrupamos los datos por **País + Sector Institucional + Tipo de Socio**.
2.  Entrenamos un modelo Prophet independiente para cada grupo.
3.  Sumamos las predicciones individuales para obtener el total del banco.

---

<a name="manos-a-la-obra"></a>
### Manos a la Obra: El Bucle de Predicción

En el Paso 3 de nuestro notebook, implementamos lo que llamo "El Bucle de la Verdad". Aquí iteramos sobre cada grupo, aislamos su historia y le pedimos a Prophet que mire 5 años hacia el futuro. Activamos explícitamente la **estacionalidad anual** (`yearly_seasonality=True`) porque los organismos multilaterales suelen tener ciclos de aprobación muy marcados (cierres fiscales, ciclos presupuestarios).

*Un vistazo simplificado al código Python que usamos:*

```python
# Iteramos sobre cada combinación única: País + Sector + Tipo de Socio
for grupo in grupos_unicos:
    # 1. Filtramos la historia específica de este grupo
    historia = df_ml[
        (df_ml["País"] == grupo.pais) & 
        (df_ml["Sector"] == grupo.sector) & 
        (df_ml["Tipo"] == grupo.tipo)
    ]
    
    # 2. Preparamos para Prophet (ds = fecha, y = monto)
    # Activamos estacionalidad anual para capturar ciclos presupuestarios
    m = Prophet(yearly_seasonality=True) 
    m.fit(historia)
    
    # 3. Predecimos 5 años (2026-2030)
    futuro = m.make_future_dataframe(periods=5, freq='Y')
    forecast = m.predict(futuro)
    
    # 4. Guardamos el resultado con sus intervalos de confianza
    predicciones.append(forecast)
```

Lo brillante de este código es que no solo nos da un número (ej. $500M), sino un **intervalo de confianza (80%)**. El modelo nos dice: *"Creo que serán 500M, pero podría ser tan bajo como 350M o tan alto como 650M"*. Para un gestor de riesgos, ese rango vale oro.

---

<a name="los-resultados"></a>
### Los Resultados: ¿Qué nos dice el 2026?

Al ejecutar nuestro modelo, Prophet nos arroja un panorama fascinante para el próximo quinquenio. El modelo ha logrado capturar las dinámicas individuales de cada socio, diferenciando claramente entre las tendencias estables de los **Socios Fundadores** y el comportamiento dinámico de los **Socios Regionales y Extraregionales**.

**¿Es plausible este crecimiento?**
Absolutamente. Si revisamos el contexto, en 2020 el banco alcanzó récords históricos de aprobación (~$3.4B) impulsados por emergencias. Con las recientes mejoras en la calificación crediticia del BCIE (alcanzando rangos de AA+ en 2025), el modelo interpreta esta capacidad de fondeo como una señal de **expansión estructural**, no solo como un pico aislado.

A continuación, presentamos la visualización consolidada. *Nota: Los amplios intervalos de confianza (banda rosa) reflejan la inherente volatilidad de las aprobaciones de desarrollo, donde factores exógenos pueden causar variaciones interanuales significativas.*

<!-- FIGURA 1: Monto Aprobado + Rango 80% + Predicción (banda desde 2025) -->
<div style="
  max-width: 840px;
  margin: 0 auto;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.06);
  padding: 14px 14px 8px 14px;
  border: 1px solid #eaeaea;
">

  <img
    src="https://quickchart.io/chart?width=820&height=420&backgroundColor=white&c={type:'line',data:{labels:['2020','2021','2022','2023','2024','2025','2026','2027','2028','2029'],datasets:[{label:'Rango 80%',data:[null,null,null,null,null,2.569,2.176,2.242,2.211,2.39],borderColor:'rgba(0,0,0,0)',backgroundColor:'rgba(255,99,132,0.28)',pointRadius:0,borderWidth:0,fill:false},{label:'',data:[null,null,null,null,null,2.569,5.663,5.707,5.584,5.793],borderColor:'rgba(0,0,0,0)',backgroundColor:'rgba(255,99,132,0.28)',pointRadius:0,borderWidth:0,fill:'-1'},{label:'Monto Aprobado',data:[3.459,3.69,4.29,3.588,2.229,2.569,null,null,null,null],borderColor:'rgb(0,0,0)',backgroundColor:'rgb(0,0,0)',borderWidth:2,pointRadius:3,lineTension:0.3,fill:false},{label:'Prediccion',data:[null,null,null,null,null,2.569,3.858,3.941,3.905,4.056],borderColor:'rgb(220,53,69)',backgroundColor:'rgb(220,53,69)',borderDash:[5,5],borderWidth:2,pointRadius:3,lineTension:0.3,fill:false}]},options:{legend:{position:'top',labels:{boxWidth:22,boxHeight:10}},title:{display:false},scales:{xAxes:[{gridLines:{display:false,drawBorder:false},ticks:{maxRotation:0,minRotation:0}}],yAxes:[{gridLines:{color:'rgba(0,0,0,0.07)',drawBorder:false},scaleLabel:{display:true,labelString:'Billones de USD'}}]}}}"
    alt="Proyección de Aprobaciones Totales BCIE con intervalo 80%"
    style="
      width: 100%;
      height: auto;
      border-radius: 10px;
      display: block;
    "
  />

  <div style="
    margin-top: 10px;
    height: 4px;
    border-radius: 999px;
    background: linear-gradient(90deg,#2BCE75,#091F5A);
    opacity: 0.7;
  "></div>

  <p style="
    text-align: center;
    font-size: 12px;
    color: #7f8c8d;
    margin: 8px 0 2px 0;
    font-family: sans-serif;
  ">
    <em>Fig 1. Histórico real (línea negra) y proyección Prophet (línea roja punteada) con intervalo de confianza 80%. Fuente: Datos históricos del Portal de Datos Abiertos del BCIE; proyecciones vía Prophet.</em>
  </p>
</div>


**El dato clave:** El modelo predice una caída abrupta para ciertos sectores privados en la región, interpretando la volatilidad de los últimos años (2023-2025) como una señal de desaceleración, mientras que el sector público mantiene una inercia de crecimiento robusta.

<a name="visualizando"></a>
### Visualizando el Futuro: El Mapa de Calor 2026-2030

Para entender la "foto completa" del quinquenio, hemos extraído los datos del modelo en una tabla consolidada. Lo que vemos es prometedor: el modelo captura y proyecta una dinámica de **crecimiento sostenido** para el BCIE a nivel global.

| Tipo de Socio | País | 2026 (Pred) | 2027 (Pred) | 2028 (Pred) | 2029 (Pred) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Fundadores** | Costa Rica | $553 M | $524 M | $657 M | $642 M |
| **Fundadores** | El Salvador | $444 M | $404 M | $427 M | $431 M |
| **Fundadores** | Guatemala | $317 M | $298 M | $363 M | $355 M |
| **Fundadores** | Honduras | $499 M | $528 M | $400 M | $425 M |
| **Fundadores** | Nicaragua | $394 M | $435 M | $366 M | $389 M |
| **SUBTOTAL** | **Fundadores** | **$2,209 M** | **$2,190 M** | **$2,215 M** | **$2,244 M** |
| Regional No Fund. | Panamá | $561 M | $605 M | $521 M | $569 M |
| Regional No Fund. | Rep. Dominicana | $253 M | $221 M | $378 M | $356 M |
| **SUBTOTAL** | **Regionales** | **$817 M** | **$827 M** | **$899 M** | **$925 M** |
| Extraregionales | Argentina | $476 M | $527 M | $482 M | $538 M |
| Extraregionales | Colombia | $355 M | $394 M | $306 M | $346 M |
| **SUBTOTAL** | **Extraregionales** | **$831 M** | **$922 M** | **$789 M** | **$885 M** |
| **TOTAL GENERAL** | | **$3,858 M** | **$3,940 M** | **$3,904 M** | **$4,055 M** |

*(Cifras en millones de USD. Fuente: Elaboración propia basada en modelo Prophet)*

**Lo que nos dicen los números:**

* **Consolidación de los Fundadores:** El bloque histórico del banco proyecta aprobaciones sólidas y constantes, manteniéndose como el motor principal de la cartera.
* **Dinámica Extraregional:** Los socios fuera de la región muestran tendencias interesantes de participación, validando la estrategia de diversificación del banco.
* **Tendencia General:** Lejos de estancarse, el modelo pronostica que el banco superará la barrera de los **$4,000 Millones** en aprobaciones totales para 2029.

---

<a name="veredicto"></a>
### El Veredicto Parcial: Luces y Sombras

Prophet ha hecho un trabajo excelente capturando la **inercia**. Ha entendido que los grandes proyectos de infraestructura pública mueven la aguja y nos ha dado un pronóstico base sólido y defendible.

**Pero... tengo una sospecha.**

Prophet es un "optimista de la tendencia". Si un país tuvo años recientes excepcionales, Prophet asume que el futuro será igual de bueno por pura inercia matemática. Pero la economía no funciona solo con inercia.

* ¿Qué pasa si cambian las tasas de interés globales?
* ¿Qué pasa si un país alcanza su techo de endeudamiento?
* ¿Prophet sabe que hubo elecciones o cambios de política monetaria? **No, no lo sabe.** Solo ve fechas y montos.

Aquí es donde nuestro "Puente al ML" se pone interesante. Necesitamos un modelo que no solo mire el calendario, sino que aprenda de las características complejas (features) del entorno económico.

**En el próximo capítulo:** Entra al ring **XGBoost (El Maestro de las Features)**. Vamos a enseñarle a nuestra IA a entender no solo *cuándo* ocurren las aprobaciones, sino *por qué*. Y veremos si logra vencer a la inercia de Prophet.

¿Crees que la inercia histórica es suficiente para predecir las finanzas de una región en desarrollo? Te leo en los comentarios.

---

***Nota Ética:** Este análisis es un ejercicio de Ciencia de Datos basado exclusivamente en datos abiertos históricos. Los pronósticos generados son estimaciones estadísticas y no constituyen asesoramiento financiero oficial ni reflejan necesariamente la política futura del BCIE. El objetivo es demostrar el potencial de estas herramientas para la planificación del desarrollo regional.*
