+++
date = "2016-08-07T20:08:11+02:00"
draft = true
title = "Análisis de datos - Mann Whitney test <Python>"
description = "Cuando no se cumplen las condiciones ANOVA podemos utilizar un Test de Mann Whitney"
slug = "ANOVA-oneway"
tags = ['Estadistica', 'python', 'Mann Whitney']
+++

El test de [Mann Whitney](https://es.wikipedia.org/wiki/Prueba_U_de_Mann-Whitney) es una herramienta [no paramétrica](https://es.wikipedia.org/wiki/Estad%C3%ADstica_no_param%C3%A9trica) que puede ejecutarse cuando las condiciones del t-test para muestras independientes fallan.

Para realizar este análisis vamos a utilizar un dataset de resultados sobre varios métodos de enseñanza de metáforas en niños.

``` python
import pandas as pd
from scipy import stats

dataset = pd.read_csv('http://www.stat.ufl.edu/~winner/data/metaphor.dat',delim_whitespace=True,header=None,usecols=[1,2])
dataset.columns= ['method','result']

# Vamos a comprobar si provienen de una distribución normalizados
# (0.793693482875824, 2.8902596227453614e-07)
# (0.7657151818275452, 1.021784257204672e-07)
```

Con los datos del test de Shapiro parece que nuestros datos no provienen de una distribución normal, vamos a recurrir a un test de Mann Whitney.

```python
stats.mannwhitneyu(dataset['method'], dataset['result'], use_continuity=False, alternative=None)
# MannwhitneyuResult(statistic=13.0, pvalue=1.3907785845023132e-19)
```

Con el resultado del test podemos rechazar la hipótesis nula sobre la similitud de medias y afirmar con una confianza del 95% (p-value < 0.05) que existe una diferencia significativa entre ambos grupos.
