+++
date = "2016-08-01T19:08:11+02:00"
draft = true
title = "Teorema central del límite"
description = "¿Qué es y para que sirve?"
slug = "teorema-central-limite"
tags = ['Estadistica', 'python', 'CLT']
+++

El teorema central del límite es uno de los teoremas más importantes en estadística y nos sirve como base de muchos métodos de análisis estadístico. Básicamente, este teorema establece que si obtenemos sucesivas muestras de una poblacion (normal o no) , las medias de estas poblaciones siguen una distribución normal.

Es decir, nos permite tratar la media de las muestras como distribuciones normales.

Como sabéis muchos procedimientos estadísticos comunes requieren que los datos provengan de distribuciones aproximadamente normales, y este teorema nos permite aplicar estos procedimientos a poblaciones que son claramente no normales.

## ¿Cómo podemos aplicar este teorema?

Básicamente si tenemos una distribución no normal(u,std). Según el teorema central del límite, si cogemos múltiples muestras de tamaño *n* y calculamos su media, estos valores tienden a formar una distribución normal(u,std/sqrt(n)).

Vamos a verlo con un ejemplo.

```python

import numpy as np
import pandas as pd
from scipy.stats import stats
import matplotlib.pyplot as plot

groupA_income = stats.norm.rvs(loc=10000, scale=2000, size=150000)
groupB_income = stats.norm.rvs(loc=40000, scale=5000, size=150000)

group_income = np.concatenate((groupA_income, groupB_income))

pd.DataFrame(group_income).describe()

# count	300000.000000
# mean	24998.062669
# std	15475.690409

pd.DataFrame(group_income).hist(bins=50)
```

![Hist](/images/7_1.png)

```python
point_estimates = []

for x in range(5000):
    sample = np.random.choice(a= group_income, size=5000)
    point_estimates.append( sample.mean() )

pd.DataFrame(point_estimates).plot(kind="density")
```

![Hist](/images/7_2.png)

```python
stats.norm.fit(point_estimates)
# (25001.345999550049, 221.37034684691793)
```
