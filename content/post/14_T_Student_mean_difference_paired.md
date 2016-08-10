+++
date = "2016-08-07T19:08:11+02:00"
draft = true
title = "Análisis de datos - Test t-student para comparar muestras dependientes <Python>"
description = "Análisis sobre cómo comparar la media de dos muestras totalmente dependientes utilizando t-student"
slug = "t-test-mean-difference-analysis-dependent"
tags = ['Estadistica', 'python', 't-student']
+++

Podemos utilizar este test para analizar dos muestras dependientes y evaluar si la media de sus valores difieren signigicativamente. Podemos por ejemplo evaluar resultados entre el mismo grupo problacional antes y después de un medicamento.

Aquí la hipótesis nula sería que no se observan direfencias significativas sobre la media y la hipótesis alternativa, generalmente con un p-value inferior al 5% significaría que las medias son diferentes.

Para este test es necesario que la diferencia de las muestras siga una distribución normal y no se presenten valores extremos significativos, por lo que tendremos que analizar esta variable antes de realizar el test.

```python

import pandas as pd
from scipy import stats

# Creamos los datos para nuestro test
pre_test_results = stats.norm.rvs(loc=6,scale=1,size=500)
post_test_results = stats.norm.rvs(loc=7.3,scale=1,size=500)
data = {'pre': pre_test_results, 'post':post_test_results}

dataset = pd.DataFrame(data)

# Calculamos la serie con la diferencia de resultados y evaluamos su normalidad
dataset['diff'] = dataset['post'] - dataset ['pre']

stats.shapiro(dataset['diff'])
# (0.9966230392456055, 0.37961140275001526)

# Ahora una vez comprobada que la diferencia de nuestras variables se distribuye bajo una distribución normal realizamos el test
stats.ttest_rel(dataset['post'],dataset['pre'])
# Ttest_relResult(statistic=-11.071961244460955, pvalue=1.2556378416021928e-25)

```

Con los resultados del test (p-value < 0.05) rechazamos la hipótesis nula y concluímos que las diferencias entre ambos test son significativamente diferentes.
