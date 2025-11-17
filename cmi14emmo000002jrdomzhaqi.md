---
title: "Explorando el Desarrollo Regional con Datos Abiertos del BCIE"
seoTitle: "Datos Abiertos BCIE y Machine Learning"
seoDescription: "Inicia una nueva serie de 'El Puente al ML'. Usamos Datos Abiertos del BCIE para analizar aprobaciones, explorar en Power BI y preparar modelos de ML."
datePublished: Sun Nov 16 2025 02:52:39 GMT+0000 (Coordinated Universal Time)
cuid: cmi14emmo000002jrdomzhaqi
slug: explorando-el-desarrollo-regional-con-datos-abiertos
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1763261358732/d99835e4-e398-4fbd-a628-0e951e788d92.png
tags: tutorial, python, data-science, machine-learning, python3, ia, open-data, power-bi, powerbi, forecasting, cienciadedatos, bcie

---

Bienvenidos a "El Puente al Machine Learning" y al inicio de nuestra primera serie de casos de estudio: **Proyectos de ML con Datos Abiertos**.

A la fecha de hoy (15 de noviembre de 2025), el portal de datos abiertos del BCIE reporta un total de **609 registros** históricos. Solo este año, ya se han aprobado **$2,328,087,479.00** en solo **16 operaciones**.

Esta es la filosofía de este blog: en lugar de datasets de juguete, usaremos estos **datos vivos** para construir modelos de Machine Learning funcionales y listos para producción.

---

### Resumen Rápido
* **Qué:** Iniciamos una serie de ML para pronosticar las aprobaciones financieras del BCIE (2026-2030).
* **Datos:** Usamos el portal oficial de Datos Abiertos del BCIE (Dataset: "Aprobaciones (General)", 609 registros, 1961-2025).
* **Cómo:** Comparamos Prophet vs. XGBoost, implementamos MLOps (v2.1) y construimos un dashboard en Power BI para explorar los datos.

---

## ¿Qué es el BCIE y por qué sus datos?

El [Banco Centroamericano de Integración Económica (BCIE)](https://www.bcie.org/) es la institución financiera multilateral de desarrollo más relevante para la región centroamericana. Fundado en 1960, su misión es promover la integración y el desarrollo económico y social de sus países miembros.

En resumen, sus decisiones tienen un impacto directo en la infraestructura, la energía, la competitividad y la salud en la región.

Recientemente, el banco ha hecho un esfuerzo ejemplar por la transparencia a través de su [Portal de Datos Abiertos](https://datosabiertos.bcie.org/). Este portal nos da acceso sin precedentes a sus operaciones, y para un científico de datos, es una mina de oro.

<a href="https://datosabiertos.bcie.org/" target="_blank" style="text-decoration: none; background-color: #f6f8fa; color: #24292e; padding: 10px 16px; border-radius: 6px; border: 1px solid #d1d5da; font-weight: 600; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji'; font-size: 14px; display: inline-block; margin: 10px 0;">
  <svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" style="vertical-align: text-bottom; margin-right: 4px; fill: currentColor;"><path d="M4.75 4.75a.75.75 0 0 0 0 1.5h6.5a.75.75 0 0 0 0-1.5h-6.5Z"></path><path d="M4.75 8a.75.75 0 0 0 0 1.5h6.5a.75.75 0 0 0 0-1.5h-6.5Z"></path><path d="M4.75 11.25a.75.75 0 0 0 0 1.5h3.5a.75.75 0 0 0 0-1.5h-3.5Z"></path><path d="M2.5 1.75C2.5.784 3.284 0 4.25 0h7.5C12.716 0 13.5.784 13.5 1.75v12.5A1.75 1.75 0 0 1 11.75 16h-7.5A1.75 1.75 0 0 1 2.5 14.25V1.75ZM4.25 1.5a.25.25 0 0 0-.25.25v12.5c0 .138.112.25.25.25h7.5a.25.25 0 0 0 .25-.25V1.75a.25.25 0 0 0-.25-.25h-7.5Z"></path></svg>
  Visitar el Portal de Datos Abiertos
</a>

## ¿A quién va dirigida esta serie?

Este es "El Puente al Machine Learning", por lo que no necesitas ser un experto para cruzarlo.

* **Si vienes de Finanzas o Economía:** Entenderás el contexto de negocio perfectamente. Te guiaremos, paso a paso, sobre cómo Prophet y XGBoost "piensan" para modelar la realidad.
* **Si vienes de Datos o Tecnología:** Entenderás los modelos. Te daremos el contexto de negocio suficiente para que sepas *por qué* una "aprobación" no es lo mismo que una "venta" y por qué la estructura de "socios" es una feature tan importante.

## Entendiendo los Datos: Socios y Aprobaciones

Antes de poder modelar, debemos entender el negocio. La estructura del BCIE es clave. Sus países miembros (socios) no se agrupan geográficamente, sino por su rol histórico y estratégico. Esta clasificación es nuestra primera *feature* fundamental:

* **Socios Fundadores:** Los cinco países que dieron origen al banco (Guatemala, El Salvador, Honduras, Nicaragua y Costa Rica).
* **Socios Regionales No Fundadores:** Países que se han incorporado posteriormente (como Panamá, República Dominicana y Belice).
* **Socios Extraregionales:** Países de fuera de la región que proveen capital y fortalecen los lazos financieros (como México, Argentina, Colombia, España, República de Corea y Taiwán).

Esta estructura define las prioridades de inversión y las líneas de financiamiento.

De todos los datasets disponibles, para este primer análisis nos enfocaremos en el más crítico para entender la estrategia del banco: las **Aprobaciones**. Una "Aprobación" es el compromiso formal del Directorio para financiar un proyecto. Es la "luz verde" que pone en marcha millones de dólares.

---

## Exploración Visual: El Dashboard Interactivo

Antes de saltar al código de Python, es fundamental "sentir" los datos. He conectado el script de Python que creamos (veremos el código en el próximo capítulo) directamente a Power BI y he publicado este dashboard interactivo.

Aquí puedes explorar los datos históricos tú mismo. Filtra por país, sector o tipo de socio para ver cómo han cambiado las tendencias a lo largo del tiempo.

<iframe title="Dashboard Ejecutivo BCIE" width="800" height="530" src="https://app.powerbi.com/view?r=eyJrIjoiNzQyODk0NmUtMjdjOC00YWY5LWFmNTgtYzc1MTMzYzM1NTA1IiwidCI6IjllMzU0NTU4LWU1YzYtNDY2NC04MzJiLTMwY2M0ODU0Yzk2NSJ9&pageName=4e8af9fd0e3c00033016" frameborder="0" allowFullScreen="true"></iframe>

---

## El Objetivo: Un Hallazgo Clave y Dos Metas

Al explorar los datos en el dashboard de arriba (específicamente filtrando desde 2010 a la fecha), emerge un hallazgo fascinante que define nuestro proyecto:

> **El *número* de aprobaciones ha venido disminuyendo desde 2010 (con un mínimo en 2024 de solo 12 operaciones), pero el *valor* de cada aprobación ha crecido exponencialmente.**

Vemos un **efecto inverso**: menos proyectos, pero mucho más grandes, alcanzando un pico histórico de monto aprobado en 2022.

<div style="
  max-width: 840px;
  margin: 0 auto;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.06);
  padding: 2px 2px 2px 2px;
">
  <iframe
    src="https://quickchart.io/chart-maker/view/zm-667062a9-03e5-4c46-82aa-0214a04416dc"
    frameborder="0"
    style="
      width: 100%;
      height: 500px;
      border: 0;
      border-radius: 18px;
      overflow: hidden;
      background: #ffffff;
    "
  ></iframe>

  <div
    style="
      margin-top: 5px;
      height: 4px;
      border-radius: 999px;
      background: linear-gradient(90deg,#2BCE75,#091F5A);
      opacity: 0.7;
    "
  ></div>
</div>





*El 'efecto inverso' en acción (2010-2025): Curvas suavizadas muestran la divergencia entre el monto aprobado (verde, eje izq) y la cantidad de operaciones (azul, eje der). Datos extraídos del portal oficial.*

Este no es un dataset estático; es un reflejo de una estrategia institucional viva. Con esto en mente, tenemos dos objetivos:

1.  **El Objetivo de Negocio (Valor Cuantificado):** ¿Podemos pronosticar esta nueva dinámica para los próximos 5 años? Un modelo preciso (ej. con un error < 15%) permitiría al banco **anticipar las necesidades de liquidez por país** o **informar al equipo de riesgo soberano** sobre la cartera de crédito proyectada.

2.  **El Objetivo de Aprendizaje (El "Puente al ML"):** Este es el corazón de nuestro blog. Ver la utilidad real de nuestras herramientas. ¿Cómo modelamos esta relación inversa? ¿Es una "tendencia" (Prophet) o un "comportamiento aprendido" (XGBoost)? Usaremos estos datos "vivos" para practicar, comparar modelos y ver cuál funciona mejor.

## Transparencia: Limitaciones y Métricas

Seremos transparentes. Los datos del mundo real son complicados.

* **Limitaciones:** Estamos viendo "Aprobaciones", no "Desembolsos". Puede haber un *rezago* significativo entre que un proyecto se aprueba y el dinero se entrega. Además, al modelar decisiones institucionales, debemos ser conscientes de que los patrones históricos pueden reflejar sesgos políticos o estructurales. **Nuestro modelo busca entender tendencias, no justificarlas.**
* **Métrica de Éxito:** Para saber qué modelo "gana", no usaremos la intuición. Nos basaremos en métricas clave como el **Error Absoluto Medio (MAE)** y el **Error Porcentual Absoluto Medio (MAPE)**.

---

## Próximos Pasos: El Desafío de Modelos

Tenemos los datos históricos y un hallazgo clave. Ahora, comienza el verdadero desafío:

**¿Cuál es el mejor modelo para pronosticar esta nueva realidad de "menos proyectos, pero más grandes"?**

En los próximos capítulos de esta serie, pondremos a competir a dos de los modelos más poderosos de la industria:

* **Capítulo 2: El Especialista en Tendencias (Prophet de Meta)**
* **Capítulo 3: El Maestro de las Features (XGBoost)**
* **Capítulo 4: El Veredicto (Prophet vs. XGBoost)**

*Spoiler: Ya hay un claro ganador en estos datos que captura mucho mejor la dinámica y entrega un mejor pronóstico. El BCIE es un organismo con mucho crecimiento y uno de los modelos hace el pronóstico a disminuir las aprobaciones, algo muy improbable y que revela una falla clave en su lógica.*

Gracias por acompañarme. ¿Tú qué modelo usarías? ¿Crees que un enfoque de series temporales o uno basado en características capturará mejor esta nueva dinámica del BCIE?

¡Déjanos tu opinión en los comentarios!
