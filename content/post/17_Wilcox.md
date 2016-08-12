+++
date = "2016-08-07T20:08:11+02:00"
draft = true
title = "Análisis de datos - Wilcox test <Python>"
description = "Cuando no se cumplen las condiciones para t-student en medidas sobre la misma muestra, podemos utilizar un Test de Wilcox"
slug = "ANOVA-oneway"
tags = ['Estadistica', 'python', 'Wilcoxon']
+++

El test de [Wilcoxon](https://es.wikipedia.org/wiki/Prueba_de_los_rangos_con_signo_de_Wilcoxon) es una herramienta [no paramétrica](https://es.wikipedia.org/wiki/Estad%C3%ADstica_no_param%C3%A9trica) que puede utilizarse como alternativa a la prueba t de Student cuando no se puede suponer la normalidad. Está basado en comparar la mediana de dos muestras **relacionadas** y determinar si existen diferencias entre ellasde dichas muestras cuando las condiciones.

Para realizar este análisis vamos a crear un juego de datos con dos distribuciones para los resultados antes y después.

``` python
import pandas as pd
from scipy import stats
import seaborn as sns
import matplotlib.pyplot as plt

%matplotlib inline

after = stats.norm.rvs(loc=10,scale=2,size=1000)
before = stats.expon.rvs(size=1000)
```

![After](/images/17_1.png)
![After](/images/17_2.png)

Obviamente fallan las condiciones para aplicar un t-test así que utilizamos el test Wilcoxon

```python
stats.wilcoxon(before,after)
# WilcoxonResult(statistic=0.0, pvalue=3.3258591189345131e-165)

np.median(after)
# 10.181853431769765

np.median(before)
# 0.72645203921873824

```

Con el resultado del test podemos rechazar la hipótesis nula y afirmar con una confianza del 95% (p-value < 0.05) que existe una diferencia significativa entre ambos grupos. Siendo la mediana del grupo *after* mayor que la inicial.
