+++
date = "2016-08-03T19:08:11+02:00"
draft = true
title = "Datos correlativos <Python>"
description = "Análisis de datos correlativos en series utilizando python"
slug = "correlation-analysis"
tags = ['Estadistica', 'python', 'Correlativos', 'pearson','spearman','kendalls']
+++

La correlación entre datos es una herramienta muy potente que nos va a permitir analizar la asociación entre nuestros datos y la intensidad y el sentido de esta asociación.

En este post vamos a partir de un ejemplo clásico (estatura - peso) para utilizar tres de los índices más comunes en el estudio de asociación de datos; Pearson, Spearman y kendall.

Existen multitud de guías sobre cuando utilizar uno u otro, quizá por las precondiciones que aplican a Pearson (normalidad, no valores extremos) este test deba ser utilizado con rangos, mientras el resto, al no implicar estas precondiciones puedan ser utilizados tanto con rangos como con valores contínuos.

Este tipo de test (descriptivos) nos van a dar una medida de "fuerza", un "sentido de la asociación" y una p-value que pruebe nuestras hipótesis.

* Vamos a obtener un valor para la asociación entre 1 y -1, siendo 1 una fuerte correlación positiva y -1 una fuerte correlación negativa. Valores cercanos a 0 implican asociación leve o no asociación, dependiendo de nuestro test estadístico (p-value).
* Vamos a obtener un valor estadístico p-value en referencia a las siguientes hipótesis:
    * H0, la asociación es 0 (no hay asociación)
    * Ha, la asociación es distinta a 0 (hay asociación)

## Pearson

El coeficiente de Pearson es una medida común para la asociación entre variables, no obstante parte de algunas condiciones que deben ser chequeadas:

* Variables normales
* Sin valores extremos significativos


```python
import pandas as pd
from scipy import stats
import matplotlib.pyplot as plt

dataset = pd.read_csv('https://vincentarelbundock.github.io/Rdatasets/csv/datasets/women.csv')

# id    0	height	weight
# 0     1	58	   115
# 1     2	59	   117
# 2     3	60	   120
# 3     4	61	   123
# 4     5	62	   126
# ...

# Probamos normalidad

stats.shapiro(dataset['height'])
# (0.963593602180481, 0.7545348405838013)

stats.shapiro(dataset['weight'])
# (0.9603573679924011, 0.6986083984375)

# Buscamos valores extremos
stats.mstats.zscore(dataset['height'],ddof=14)
# array([-0.41833001, -0.35856858, -0.29880715, -0.23904572, -0.17928429,
#        -0.11952286, -0.05976143,  0.        ,  0.05976143,  0.11952286,
#         0.17928429,  0.23904572,  0.29880715,  0.35856858,  0.41833001])
stats.mstats.zscore(dataset['weight'],ddof=14)
# array([-0.37477207, -0.34028384, -0.2885515 , -0.23681916, -0.18508682,
#        -0.13335448, -0.08162214, -0.0298898 ,  0.03908666,  0.090819  ,
#         0.15979545,  0.2287719 ,  0.29774836,  0.38396893,  0.47018949])

# Una vez comprobadas las condiciones hacemos el test
stats.pearsonr(dataset['height'],dataset['weight'])
# (0.99549476778421619, 1.0909729585997641e-14)
```

Con los resultados del test podemos concluir:

* Nuestro p-value es muy bajo, inferior al 99% (two tailed) así que podemos rechazar la hipótesis nula (H0: No hay asociación)
* Nuestro coeficiente de asociación es positivo (a medida que crece / decrece una variable la otra lo hace en el mismo sentido) y muy cercano a 1, lo que muestra una asociación muy relevante.

## Spearman && Kendall

El test de correlación de Spearman es una versión del test de Pearson basada en rangos, lo que nos permite evitar las condiciones anteriores de normalidad y valores extremos.

```python
import pandas as pd
from scipy import stats
import matplotlib.pyplot as plt

# Utilizamos ahora un conjunto de datos no normal
dataset = pd.read_csv('https://vincentarelbundock.github.io/Rdatasets/csv/car/Davis.csv',usecols=['sex','height','weight'])

stats.spearmanr(dataset['height'],dataset['weight'])
# SpearmanrResult(correlation=0.76843051763130032, pvalue=3.0242973362885107e-40)
```

El test de Kendall utiliza un método diferente, calculando las diferencias entre los posibles pares de datos. Su aplicación en cambio es muy similar.

```python
import pandas as pd
from scipy import stats
import matplotlib.pyplot as plt

# Utilizamos ahora un conjunto de datos no normal
dataset = pd.read_csv('https://vincentarelbundock.github.io/Rdatasets/csv/car/Davis.csv',usecols=['sex','height','weight'])
stats.kendalltau(dataset['height'],dataset['weight'])
```

Tenéis bastante información sobre las diferencias [aquí](http://d-scholarship.pitt.edu/8056/1/Chokns_etd2010.pdf).
