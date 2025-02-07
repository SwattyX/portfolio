---
title: "Análisis de Sentimiento sobre la Vacuna del Covid-19 en Twitter"
date: 2022-08-05
description: "Análisis de la opinión pública sobre las vacunas del Covid-19 utilizando NLP y análisis de sentimiento."
tags: ["NLP", "Sentiment Analysis", "Twitter Data", "Covid-19"]
showDate: true
showWordCount: true
draft: false

---

![alt text](twitter-covid-vaccines-nlp.jpg)

# **Introducción**

Las redes sociales se han convertido en una plataforma clave para discutir temas globales, y la pandemia de Covid-19 no fue la excepción. Millones de usuarios compartieron sus opiniones sobre las vacunas del Covid-19 en Twitter, que iban desde la aprobación absoluta hasta el escepticismo y la desinformación. Para comprender mejor estas opiniones, realicé un **Análisis de Sentimiento sobre las vacunas del Covid-19 en Twitter** utilizando **Procesamiento de Lenguaje Natural (NLP)** como parte de un trabajo durante mi licenciatura.

Este proyecto tenía como objetivo explorar cómo reaccionó el público ante las vacunas del Covid-19 a lo largo del tiempo, qué vacunas fueron más favorecidas y cómo la desinformación jugó un papel importante en la formación de las discusiones. En este blog, te guiaré a través del **proceso de recolección de datos, las técnicas de análisis de sentimiento y los conocimientos clave** obtenidos a partir de más de **614,000 tweets** relacionados con las vacunas del Covid-19.

---

# **Recolección de Datos y Preprocesamiento**

## **1. Fuente de los Datos**
El conjunto de datos fue obtenido de **Kaggle**, que contenía tweets sobre las vacunas del Covid-19 recolectados por diferentes usuarios. El conjunto de datos consistió en dos fuentes principales:

- **Tweets sobre la Vacuna Covid**
- **Tweets sobre Todas las vacunas del Covid-19-19**

Estos conjuntos de datos fueron fusionados, lo que resultó en un conjunto de datos final de **614,074 tweets** que abarcan desde **enero de 2020 hasta abril de 2022**. El conjunto de datos proporcionó una visión amplia del sentimiento público a lo largo de diferentes etapas de la pandemia, incluyendo el desarrollo de vacunas, aprobaciones y distribuciones.

<div style="overflow-x: auto;">

<table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>id</th><th>user_name</th><th>user_location</th><th>user_description</th><th>user_created</th><th>user_followers</th><th>user_friends</th><th>user_favourites</th><th>user_verified</th><th>date</th><th>text</th><th>hashtags</th><th>source</th><th>retweets</th><th>favorites</th><th>is_retweet</th></tr></thead><tbody><tr><th>0</th><td>1340539111971516416</td><td>Rachel Roh</td><td>La Crescenta-Montrose, CA</td><td>Aggregator of Asian American news; scanning diverse sources 24/7/365. RT's, Follows and 'Likes' will fuel me 👩‍💻</td><td>2009-04-08 17:52:46</td><td>405</td><td>1692</td><td>3247</td><td>False</td><td>2020-12-20 06:06:44</td><td>Same folks said daikon paste could treat a cytokine storm #PfizerBioNTech https://t.co/xeHhIMg1kF</td><td>['PfizerBioNTech']</td><td>Twitter for Android</td><td>0</td><td>0</td><td>False</td></tr><tr><th>1</th><td>1338158543359250433</td><td>Albert Fong</td><td>San Francisco, CA</td><td>Marketing dude, tech geek, heavy metal &amp; '80s music junkie. Fascinated by meteorology and all things in the cloud. Opinions are my own.</td><td>2009-09-21 15:27:30</td><td>834</td><td>666</td><td>178</td><td>False</td><td>2020-12-13 16:27:13</td><td>While the world has been on the wrong side of history this year, hopefully, the biggest vaccination effort we've ev… https://t.co/dlCHrZjkhm</td><td>NaN</td><td>Twitter Web App</td><td>1</td><td>1</td><td>False</td></tr></tbody></table></div>

## **2. Pasos de Preprocesamiento**

Antes de aplicar el análisis de sentimiento, los datos pasaron por un extenso **proceso de limpieza y transformación** para eliminar el ruido y estandarizar el texto para su análisis. Los siguientes pasos fueron implementados:

- **Eliminación de tweets duplicados y contenido generado por bots**: Para evitar sesgar los resultados.
- **Eliminación de URLs, menciones y hashtags** para centrarse solo en el contenido textual.
- **Tokenización**: Dividir las oraciones en palabras individuales.
- **Lematización**: Convertir las palabras a sus formas base (por ejemplo, "corriendo" → "correr").
- **Eliminación de palabras vacías**: Filtrar palabras comunes como "el," "y," y "es" que no contribuyen al sentimiento.
- **Manejo de caracteres especiales y emojis**: Convertir los emojis en representaciones de texto para conservar el sentimiento.

Para estas tareas, se utilizaron bibliotecas de Python como **TextBlob, NLTK, pandas y NeatText**. El objetivo era crear un conjunto de datos que reflejara con precisión el sentimiento humano sin que los puntos de datos irrelevantes afectaran los resultados. Después de limpiar los datos, el conjunto resultante consistió en **482,523 tweets**.

![alt text](image.png)

---

# **Metodología de Análisis de Sentimiento**

## **1. Clasificación de Sentimiento**

Cada tweet fue clasificado en una de tres categorías de sentimiento:

- **Positivo**: Opiniones favorables sobre las vacunas del Covid-19.
- **Neutral**: Tweets informativos o no opuestos.
- **Negativo**: Escepticismo, desinformación o desconfianza hacia las vacunas.

Esta clasificación se realizó utilizando **TextBlob**, una biblioteca de Python que asigna **puntajes de polaridad** al texto:

- **Polaridad** varía de **-1 (negativo) a +1 (positivo)**.
- Un puntaje de polaridad **>0** se considera positivo, **<0** negativo y **0** neutral.

## **2. Análisis de Subjetividad**

También se midió la **subjetividad**, que determina cuán factual frente a opnativo es un tweet. Los puntajes de subjetividad ayudaron a distinguir entre reportes de noticias fácticos y opiniones personales, permitiendo ver cuánto del discurso sobre las vacunas se basó en emociones en lugar de hechos verificables.

## Análisis de Sentimiento con TextBlob

Esta función de Python utiliza la biblioteca TextBlob para analizar el sentimiento de un texto dado. Retorna un diccionario que contiene la polaridad, subjetividad y la clasificación general del sentimiento del texto.

```python
from textblob import TextBlob

def analyze_sentiment(text):

    analysis = TextBlob(text)
    polarity = analysis.sentiment.polarity
    subjectivity = analysis.sentiment.subjectivity

    if polarity > 0:
        sentiment = 'Positive'
    elif polarity == 0:
        sentiment = 'Neutral'
    else:
        sentiment = 'Negative'

    result = {
        'polarity': polarity,
        'subjectivity': subjectivity,
        'sentiment': sentiment
    }

    return result
```
---

# **Hallazgos Clave**

## **1. Distribución General del Sentimiento**

![alt text](image-3.png)

El conjunto de datos mostró la siguiente distribución de sentimientos:

- **42.6% Positivo**
- **43.8% Neutral**
- **13.6% Negativo**

Esto indica que, aunque la mayoría de los tweets fueron neutrales, el sentimiento positivo hacia las vacunas superó ligeramente al sentimiento negativo. Este es un hallazgo alentador, que muestra que, a pesar de la vacilación hacia las vacunas y la desinformación, los usuarios de las redes sociales fueron en su mayoría solidarios o al menos informativos sobre las vacunas.

## **2. Sentimiento Específico por Vacuna**

Los puntajes de sentimiento para diferentes vacunas fueron los siguientes:

| Vacuna     | Polaridad | Subjetividad |
| ----------- | -------- | ------------ |
| Pfizer      | 0.1163   | 0.3176       |
| AstraZeneca | 0.114    | 0.2685       |
| Sputnik     | 0.1082   | 0.3041       |
| Covaxin     | 0.1080   | 0.2541       |
| Moderna     | 0.1047   | 0.2954       |

- **Pfizer** tuvo la mayor aceptación basada en la polaridad.
- **Moderna** tuvo la polaridad más baja, pero aún estaba **por encima de 0**, indicando un sentimiento positivo en general.
- **Covaxin** tuvo la subjetividad más baja, lo que significa que se hicieron más declaraciones objetivas sobre ella.

Estos resultados reflejan cómo diferentes vacunas fueron recibidas por el público y proporcionan información sobre la confianza y percepción de la marca.


## **3. Análisis de Series Temporales de Sentimiento**

![alt text](image-4.png)

![alt text](image-5.png)

El análisis del sentimiento a lo largo del tiempo reveló tendencias clave:

- A principios de **2020** hubo **baja actividad en los tuits** sobre las vacunas debido a la falta de información disponible.
- El sentimiento **aumentó en diciembre de 2020**, coincidiendo con el lanzamiento de la **vacuna de Pfizer** bajo autorización de uso de emergencia (EUA).
- El **mayor pico en el sentimiento ocurrió en agosto de 2021**, coincidiendo con la **aprobación de la tercera dosis** en EE. UU.

## **4. Palabras Más Comunes en las Categorías de Sentimiento**

Utilizando **nubes de palabras**, identificamos las palabras más frecuentemente utilizadas en diferentes categorías de sentimiento:

### **Palabras Positivas:**
- Vacuna, Eficiente, Agradecido, Seguro, Increíble, Voluntario

### **Palabras Negativas:**
- Peligroso, Asustado, Desinformación, Efectos secundarios, Arriesgado

### **Palabras Neutras:**
- Vacuna, Dosis, Salud, Disponible, Anuncio

---

# **Desafíos y Limitaciones**

Aunque el análisis proporcionó perspectivas valiosas, también enfrentó algunas limitaciones:

- **Sesgo en los Datos de Twitter**: El conjunto de datos puede no representar la opinión de la población global.
- **Detección de Ironía y Sarcasmo**: Algunos tuits con sarcasmo pueden haber sido mal clasificados.
- **Tuits Generados por Bots**: A pesar de la filtración, algunos tuits automatizados podrían haber influido en los resultados.

---

# **Conclusión y Lecciones Aprendidas**

Este proyecto proporcionó una **perspectiva basada en datos sobre el sentimiento público hacia las vacunas contra el Covid**, destacando tendencias clave y reacciones. Las principales lecciones aprendidas son:

- El sentimiento público fue **en su mayoría neutral a positivo**.
- **Pfizer** tuvo la percepción más positiva entre las vacunas.
- El sentimiento **aumentó durante los hitos clave de aprobación de las vacunas**.

Entender la opinión pública es crucial para **campañas de salud pública**, combatir la desinformación y mejorar las estrategias de distribución de vacunas. Las mejoras futuras podrían incluir **modelos de sentimiento basados en deep learning** y **análisis en tiempo real de la percepción de las vacunas.**

---

¡Gracias por leer! Si tienes alguna pregunta o comentario, no dudes en contactarme. Aprecio mucho tu retroalimentación.

---

**Keywords:** NLP, Sentiment Analysis, Covid, Machine Learning, Data Science

---