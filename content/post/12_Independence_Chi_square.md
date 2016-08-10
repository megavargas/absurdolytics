+++
date = "2016-08-05T19:08:11+02:00"
draft = true
title = "Análisis de datos - Test chi-cuadrado de independencia de variables <Python>"
description = "Análisis sobre la dependencia de variables utilizando Chi-cuadrado"
slug = "chi-independence-analysis"
tags = ['Estadistica', 'python', 'Asociación']
+++

El test chi-cuadrado también nos sirve para analizar si dos variables categóricas son independientes. Est test puede ser utilizado para determinar si variables como educación, política, etc varían en base a parámetros demográficos como género, religión etc.

Nosotros vamos a hacer una prueba con alimentación de animales, midiendo el peso de diferentes animales en función de su alimentación.


```python

import pandas as pd
dataset = pd.read_csv('https://vincentarelbundock.github.io/Rdatasets/csv/datasets/chickwts.csv',usecols=['weight','feed'])

"""
weight	feed	weight_rank
0	179	horsebean	200.0
1	160	horsebean	200.0
2	136	horsebean	150.0
...
"""

# Vamos a aplicar una función para ranquear los datos y hacer una tabla cruzada con las frecuencias de cada ocurrencia

def get_ranks(data):
    if (data < 150):
        return 150
    elif (150 < data < 200):
        return 200
    elif (200 < data < 250):
        return 250
    elif (250 < data < 300):
        return 300
    elif (300 < data < 350):
        return 350
    elif (350 < data):
        return 400

dataset['weight_rank'] = dataset['weight'].map(get_ranks)
ct_dataset = pd.crosstab(dataset['feed'],dataset['weight_rank'], dropna=True)

"""
weight_rank	150.0	200.0	250.0	300.0	350.0	400.0
feed						
casein	        0	0	2	2	2	6
horsebean	5	3	2	0	0	0
linseed	        2	2	4	3	1	0
meatmeal	0	1	2	3	4	1
soybean	        0	4	4	2	3	0
sunflower	0	0	1	2	7	2
"""

import scipy.stats
scipy.stats.chi2_contingency(ct_dataset)

"""
(62.52618759236406,
 4.649385714865837e-05,
 25,
 array([[ 1.2       ,  1.71428571,  2.57142857,  2.05714286,  2.91428571, 1.54285714],
        [ 1.        ,  1.42857143,  2.14285714,  1.71428571,  2.42857143, 1.28571429],
        [ 1.2       ,  1.71428571,  2.57142857,  2.05714286,  2.91428571, 1.54285714],
        [ 1.1       ,  1.57142857,  2.35714286,  1.88571429,  2.67142857, 1.41428571],
        [ 1.3       ,  1.85714286,  2.78571429,  2.22857143,  3.15714286, 1.67142857],
        [ 1.2       ,  1.71428571,  2.57142857,  2.05714286,  2.91428571, 1.54285714]]))
"""

```

Viendo nuestro *p.value* inferior al 5% podemos rechazar la hipótesis nula (son variables independientes) y aceptar la hipótesis nula sobre su dependencia.
