+++
date = "2016-07-03T19:08:11+02:00"
draft = true
title = "Teorema central del límite"
description = "Qué implicay cómo se aplica este teorema."
slug = "teorema-central-limite"
tags = ['estadística']
+++

Para completar el [post anterior](post/distribuciones-probabilidad-scipy) sobre distribuciones de probabilidad es necesario que hablemos sobre el teorema central del límite.

Citando a [wikipedia](https://es.wikipedia.org/wiki/Teorema_del_l%C3%ADmite_central) "El teorema del límite central o teorema central del límite indica que, en condiciones muy generales, si Sn es la suma de n variables aleatorias independientes y de varianza no nula pero finita, entonces la función de distribución de Sn «se aproxima bien» a una distribución normal (también llamada distribución gaussiana, curva de Gauss o campana de Gauss). Así pues, el teorema asegura que esto ocurre cuando la suma de estas variables aleatorias e independientes es lo suficientemente grande."

En la práctica, lo que quiere decir este teorema es que si tenemos un gran conjunto de datos y obtenemos muestras aleatorias de este, las medias de estas muestras siguen una distribución normal con la misma media y una desviación típica igual al error estandar.

Esto nos permite responder a problemas del siguiente tipo.

> Con los datos de uso ocasional de BiciMad, ¿Cúal es el intervalo de confianza para la media de uso diario si cogemos un periódo de una semana de uso ?

Vamos a resolver este problema con scipy

```pyt
import numpy as np
import pandas as pd
import scipy.stats as st
import scipy

data = pd.read_csv('http://datos.madrid.es/egob/catalogo/216343-0-bicimad-disponibilidad.csv',
    sep=';', thousands='.', decimal=',').dropna()

uso_ocasional = data['USOS_ABONADO_OCASIONAL']

mean = uso_ocasional.mean()
std  = uso_ocasional.std()

# Nueva desviación estándar con una muestra de 7 días
ustd = std / 7

# Cálculo del intervalo de confianza para la media con la nueva desviación estándar
st.norm.interval(0.95,mean,ustd)
# (189.75358701949591, 222.22156826000719)

```

Este teorema es realmente útil y tiene inumerables usos, puedes obtener más información aquí
