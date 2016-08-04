+++
date = "2016-08-01T19:08:11+02:00"
draft = true
title = "Análisis de una variable - T student"
description = "Análisis de una variable, la distribución t-student y su aplicación a través del teorema central del límite"
slug = "onevariate-t-student-clt"
tags = ['Estadistica', 'python', 't-student', 'CLT']
+++

La distribución t de Student se suele utilizar para hacer estimaciones de la media cuando se desconoce la varianza (es lo habitual) y se usan muestas pequeñas.

Podríamos analizar por ejemplo si una muestra obtenida de una población tiene las mismas características que la población, o evaluar la diferencia entre dos muestras.

Para ser utilizada se deben dar las siguientes condiciones:

* La variable a estudiar está normalmente distribuida
* La variable a estudiar no presenta muchos valores extremos

## Carga inicial de datos y comprobación de hipótesis

Para este ejemplo vamos a partir de un conjunto de datos del portal de datos abiertos de Madrid sobre el uso de BiciMad, concretamente sobre el parámetro HORAS_TOTALES_USOS_BICICLETAS.

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from scipy import stats

data = pd.read_csv('http://datos.madrid.es/egob/catalogo/216343-0-bicimad-disponibilidad.csv', sep=';', thousands='.', decimal=',').dropna()

data['HORAS_TOTALES_USOS_BICICLETAS'].describe()

# count     343.000000
# mean     3708.201574
# std      1231.080804
# min       759.880000
# 25%      2906.215000
# 50%      3552.050000
# 75%      4531.785000
# max      6680.650000

sp.stats.shapiro(data['HORAS_TOTALES_USOS_BICICLETAS'])
# (0.9809087514877319, 0.0001618135574972257)
```

Lo primero que podemos observar es que nuestra distribución no es normal, por lo que en principio no podríamos utilizar el t-student; Aquí entra el teorema central del límite.

## Teorema central del límite

El teorema central del límite establece que en condiciones generales, si Sn es la suma de n variables aleatorias independientes y de varianza no nula pero finita, entonces la función de distribución de Sn «se aproxima bien» a una distribución normal.

Este teorema es muy importante, ya que muchos procedimientos estadísticos comunes requieren que los datos provengan de distribuciones aproximadamente normales, y este teorema nos permite aplicar estos procedimientos a poblaciones que son claramente no normales.

¿Cómo podemos aplicar este teorema?

Básicamente ahora tenemos una distribución claramente no normal(u,std). Según el teorema si cogemos muestras de tamaño *n* y calculamos su media, en el límite tenderemos a una distribución que aproxima a una distribución normal(u,std/sqrt(n)).

Esto nos permite responder preguntas del tipo:

* ¿Qué intervalo medio de uso podemos esperar en 30 días con un intervalo de confianza del 95%?
```python
stats.t.interval(0.95, df = 29, loc = 3688.003445, scale= (1214.821045/np.sqrt(30)))     
#(3234.381811734112, 4141.6250782658881)
```

* ¿Qué probabilidad tenemos de tener una media de 5000 horas en el próximo mes?
```python
#stats.ttest_1samp(data['HORAS_TOTALES_USOS_BICICLETAS'], 5000)
Ttest_1sampResult(statistic=-20.405840879535834, pvalue=7.6100510234310873e-62)
```
