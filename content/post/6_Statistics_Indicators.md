+++
date = "2016-07-30T19:08:11+02:00"
draft = true
title = "Análisis de datos - Normalidad y Valores extremos <Python>"
description = "Test de normalidad, valores estadísticos básicos y obtención de valores extremos"
slug = "Estadistica-basica-python"
tags = ['Estadistica', 'python']
+++

En este post vamos a partir de un conjunto de datos del portal de datos abiertos de Madrid sobre el uso de BiciMad, concretamente sobre el parámetro HORAS_TOTALES_USOS_BICICLETAS sobre el que vamos a obtener algunos parámetros básicos estadísticos, ver si podemos ajustar los datos a una distribución normal y también vamos a obtener los valores externos.

## Carga de datos

```python
import numpy as np
import pandas as pd
import scipy as sp

data = pd.read_csv('http://datos.madrid.es/egob/catalogo/216343-0-bicimad-disponibilidad.csv', sep=';', thousands='.', decimal=',').dropna()
```

## Obtención de parámetros estadísticos básicos

```python
data['HORAS_TOTALES_USOS_BICICLETAS'].describe()

# count     343.000000
# mean     3708.201574
# std      1231.080804
# min       759.880000
# 25%      2906.215000
# 50%      3552.050000
# 75%      4531.785000
# max      6680.650000
# Name: HORAS_TOTALES_USOS_BICICLETAS, dtype: float64
```

## Skewness - Kurtosis

El valor Skewness nos informa sobre la falta de simetría en un conjunto de datos:

* Si el valor es cero o muy cercano a cero nos indica simetría
* Si el valor es positivo nos indica que hay más valores hacia la derecha y viceversa
* Generalmente un valor mayor que uno (o menor que -1) nos indica que la distribución está lejos de ser simétricas

```python
data['HORAS_TOTALES_USOS_BICICLETAS'].skew()
# 0.3166993522734674
```

El valor kurtosis nos informa como están distribuidos los valores y su semejanza con una distribución gausiana:

* Una distribución "plana" tendrá un valor de kurtosis negativo
* Una distribución donde los valores se concentren en el centro tendrá un valor de kurtosis positivo

```python
data['HORAS_TOTALES_USOS_BICICLETAS'].kurtosis()
# -0.46370831811496371
```

## Test de normalidad

Vamos a utilizar el test de [shapiro will](https://es.wikipedia.org/wiki/Test_de_Shapiro%E2%80%93Wilk) para analizar la normalidad de los datos.

```python
sp.stats.shapiro(data['HORAS_TOTALES_USOS_BICICLETAS'])
# (0.9809087514877319, 0.0001618135574972257)
```

Este test tiene como hipótesis nula que nuestros datos se ajustan a una distribución normal y una hipotesis alternativa que rechaza esta asunción. Aquí debemos fijarnos en el segundo parámetro que nos el intervalo de confianza. Podemos ver que es inferior al 95%
por lo que rechazamos la hipótesis nula.

A parte de este método numérico existe la posibilidad de utilizar métodos más visuales, no obstante son susceptibles de distintas interpretaciones.

![ELK](/images/5_4.png)
![ELK](/images/5_5.png)

## Valores extremos

Para calcular los días con valores extremos vamos a añadir una columna con el [número de desviaciones estándares sobre el valor medio](https://en.wikipedia.org/wiki/Standard_score) y vamos a filtrar por aquellos mayores / menores de 2.

```python
data['Z-SCORE'] = stats.mstats.zscore(service_hours_data,ddof=1)

data[data['Z-SCORE']>2]['DIA ']
# 32    11/09/2015
# 41    20/09/2015
# 46    25/09/2015
# 47    26/09/2015
# 48    27/09/2015
# 53    02/10/2015
# 54    03/10/2015
# 60    09/10/2015

data[data['Z-SCORE']<-2]['DIA ']
# 107    25/11/2015
# 238    04/04/2016
# 246    12/04/2016
# 272    08/05/2016
```
