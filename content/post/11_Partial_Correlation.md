+++
date = "2016-08-03T19:08:11+02:00"
draft = true
title = "Análisis de datos - Asociación parcial de datos <Python/R>"
description = "Análisis de datos correlativos parciales utilizando python y rpy2"
slug = "partial-correlation-analysis"
tags = ['Estadistica', 'python', 'Asociación']
+++

En este post vamos a ver un ejemplo clásico de asociación parcial y como eliminar de nuestra asociación ruidos que nos puedan dar resultados erróneos.

Voy a aprovechar para introducir [rpy2](rpy2.bitbucket.org), un proyecto que nos permite interactuar con las librerías de R desde Python. El motivo es que no he encontrado ningún método rápido para este tipo de análisis en python y ya de paso vemos como podemos integrar ambas tecnologías.

El ejemplo a utilizar es temperatura - consumo de helados x habitante - insolaciones sufridas x 1000 habitantes, una tabla sencilla que vamos a cargar diréctamente en un DataFrame de pandas.

```python

import rpy2.interactive as r

# Vamos a incorporar pandas2ri para interactuar fácilmente entre los datos de pandas y R
from rpy2.robjects import pandas2ri
pandas2ri.activate()

# Ahora cargamos el paquete ppcor para el análisis parcial en asociaciones de datos
from rpy2.robjects.packages import importr
ppcor =importr('ppcor')

import pandas as pd

data = [
    [16,0,0],
    [17,0,12],
    [19,1,24],
    [21,1,27],
    [22,2,32],
    [24,4,64],
    [28,5,51],
    [33,7,45],
    [28,4,37],
    [23,2,21],
    [19,1,11],
    [16,0,0]
]

dataframe = pd.DataFrame(data)
dataframe.columns = ['temperatura','helados_persona','insolaciones']

from scipy import stats

# Vamos a utilizar Pearson así que primero vamos a probar normalidad y valores extremos
stats.shapiro(dataframe['temperatura'])
# (0.9291362166404724, 0.3710464537143707)

stats.shapiro(dataframe['helados_persona'])
# (0.8792054653167725, 0.08561689406633377)

stats.shapiro(dataframe['insolaciones'])
# (0.9665603041648865, 0.8716508150100708)

stats.mstats.zscore(dataframe['temperatura'],ddof=11)
# array([-0.34819   , -0.29172675, -0.17880027, -0.06587378, -0.00941054,
#         0.10351594,  0.32936892,  0.61168513,  0.32936892,  0.0470527 ,
#        -0.17880027, -0.34819   ])

stats.mstats.zscore(dataframe['helados_persona'],ddof=11)
# array([-0.3       , -0.3       , -0.16666667, -0.16666667, -0.03333333,
#         0.23333333,  0.36666667,  0.63333333,  0.23333333, -0.03333333,
#        -0.16666667, -0.3       ])

stats.mstats.zscore(dataframe['insolaciones'],ddof=11)
# array([-0.40806175, -0.22670097, -0.04534019,  0.        ,  0.07556699,
#         0.55919573,  0.36272155,  0.27204117,  0.15113398, -0.09068039,
#        -0.24181437, -0.40806175])

stats.pearsonr(dataframe['helados_persona'],dataframe['insolaciones'])
# (0.83224445382488932, 0.0007837572449094477)
```

Según el test de pearson hay una fuerte relación entre los helados consumidos por cada persona y las insolaciones sufridas, lo que a priori parece absurdo. Vamos ahora a examinar la misma relación eliminando el factor temperatura.

```python
ppcor.pcor_test(dataframe['helados_persona'],dataframe['insolaciones'],dataframe['temperatura'], method="pearson")

"""
R object with classes: ('data.frame',) mapped to:
<DataFrame - Python:0x1207d5d40 / R:0x10f5a7670>
[Float..., Float..., Float..., IntVe..., Float..., Facto...]
  estimate: <class 'rpy2.robjects.vectors.FloatVector'>
  R object with classes: ('numeric',) mapped to:
<FloatVector - Python:0x1207d5b00 / R:0x10f5a4b78>
[0.453578]
  p.value: <class 'rpy2.robjects.vectors.FloatVector'>
  R object with classes: ('numeric',) mapped to:
<FloatVector - Python:0x1207d5b48 / R:0x10f5a4a28>
[0.161149]
  statistic: <class 'rpy2.robjects.vectors.FloatVector'>
  R object with classes: ('numeric',) mapped to:
<FloatVector - Python:0x1207d5998 / R:0x10f5a48d8>
[1.526828]
  n: <class 'rpy2.robjects.vectors.IntVector'>
  R object with classes: ('integer',) mapped to:
<IntVector - Python:0x1207d58c0 / R:0x10f590a28>
[      12]
  gp: <class 'rpy2.robjects.vectors.FloatVector'>
  R object with classes: ('numeric',) mapped to:
<FloatVector - Python:0x1207d57e8 / R:0x10f5909c8>
[1.000000]
  Method: <class 'rpy2.robjects.vectors.FactorVector'>
  R object with classes: ('factor',) mapped to:
<FactorVector - Python:0x1207d5560 / R:0x10f5d1b48>
[       1]
"""
```

Entre estos números e información podemos distingir que el parámetro *estimate* ha descendido a 0.45 y el *p.value* ya no es suficiente para rechazar la hipótesis nula y por lo tanto, no hay una asociación.
