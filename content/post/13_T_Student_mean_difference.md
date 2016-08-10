+++
date = "2016-08-06T19:08:11+02:00"
draft = true
title = "Análisis de datos - Test t-student para comparar muestras independientes <Python>"
description = "Análisis sobre cómo comparar la media de dos muestras totalmente independientes utilizando t-student"
slug = "t-test-mean-difference-analysis"
tags = ['Estadistica', 'python', 't-student']
+++

Podemos utilizar este test para analizar dos muestras independentes y evaluar si la media de sus valores difieren signigicativamente. Podemos por ejemplo evaluar resultados entre distintos grupos poblacionales (no para probar sobre un mismo grupo por ejemplo antes y después de un medicamento)

Aquí la hipótesis nula sería que no se observan direfencias significativas sobre la media y la hipótesis alternativa, generalmente con un p-value inferior al 5% significaría que las medias son diferentes.

Para este test es necesario que las muestras sigan una distribución normal y es importante que evaluemos la diferencia de sus varianzas, ya que si no son similares utilizaremos una variante denominada [Welch t-test](https://en.wikipedia.org/wiki/Welch%27s_t-test).

```python

import pandas as pd
from scipy import stats

# Cargamos un dataset con resultados de sueño sobre dos grupos de alumnos
dataset = pd.read_csv('https://vincentarelbundock.github.io/Rdatasets/csv/datasets/sleep.csv')

"""
    extra	group
0	0.7	1
1	-1.6	1
2	-0.2	1
3	-1.2	1
4	-0.1	1
5	3.4	1
6	3.7	1
7	0.8	1
8	0.0	1
9	2.0	1
10	1.9	2
11	0.8	2
12	1.1	2
13	0.1	2
14	-0.1	2
15	4.4	2
16	5.5	2
17	1.6	2
18	4.6	2
19	3.4	2
"""

# Ahora vamos a probar que ambas variables son normales con el test de shapiro

stats.shapiro((dataset[dataset['group']==1])['extra'])
# (0.9258062243461609, 0.40793028473854065)

stats.shapiro((dataset[dataset['group']==2])['extra'])
# (0.9192977547645569, 0.35113534331321716)

```

Ambos p-value son mayores de 0.05 así que aceptamos la hipótesis nula sobre su normalidad.

Para probar la similitud de varianzas utilizaremos el test de [Levene](https://es.wikipedia.org/wiki/Prueba_de_Levene) que nos va a decir si tienen la misma varianza (hipótesis nula, p-value > 0.05) o en cambio sus varianzas son significativamente diferentes (hipótesis alternativa, p-value < 0.05).

```python
stats.levene(dataset['extra'],dataset['group'])
# LeveneResult(statistic=14.96991395012876, pvalue=0.00041556687750914014)
```

Podemos ver que tienen una varianza significativamente diferente, y por tanto debemos especificarlo al hacer nuestro t-test.

```python
stats.ttest_ind(dataset['extra'],dataset['group'], equal_var = False)
#Ttest_indResult(statistic=0.085915699272644433, pvalue=0.93233052440188924)
```

Podemos concluir con los valores de p-value que no hay diferencias significativas en las muestras de sueño de los dos diferentes grupos.
