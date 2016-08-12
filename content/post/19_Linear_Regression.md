+++
date = "2016-08-10T20:08:11+02:00"
draft = true
title = "Análisis de datos - Regresión Lineal<Python>"
description = "Ejemplo de Regresión lineal con python y scikit-learn"
slug = "Linear-Regression"
tags = ['Estadistica', 'python', 'Linear-Regression']
+++

Este método, al igual que los métodos para evaluar la correlación entre variables mide la relación entre una variable dependiente y una variable independiente, en cambio aquí no vamos a obtener un valor de correlación sino un modelo a través del cual podemos predecir nuevos valores.

La regresión lineal es el método más simple de regresión. En este post vamos a utilizar el método de [mínimos cuadrados](https://es.wikipedia.org/wiki/M%C3%ADnimos_cuadrados) que intenta encontrar la función *lineal* que mejor se aproxima los datos de acuerdo con el criterio de mínimo error cuadrático.

Antes de comenzar debemos fijarnos en las condiciones a chequear antes de simplemente crear nuestro modelo:

* La relación entre variables es lineal.
* Las variables no presentan valores extremos importantes.
* Hay independencia de errores.
* No hay [multicolinealidad](https://es.wikipedia.org/wiki/Multicolinealidad) (o es pequeña).
* Existe [homocedasticidad](https://es.wikipedia.org/wiki/Homocedasticidad).
* Los [valores residuales](https://en.wikipedia.org/wiki/Errors_and_residuals) se distribuyen normalmente.

Para este post vamos a utilizar un dataset de concentración de la droga LSD y los resultados de un test. Utilizaremos python y scikit-learn.

```python
import pandas as pd
from scipy import stats
from sklearn import linear_model
import seaborn as sns
import numpy as np
import matplotlib.pyplot as plt
from sklearn.cross_validation import train_test_split

%matplotlib inline

# Cargamos los datos
dataset = pd.read_csv('http://www.stat.ufl.edu/~winner/data/lsd.dat',
                      delim_whitespace=True, header=None)
dataset.columns = ['drug_concentration','score']
```

## La relación entre variables es lineal y no se presentan valores extremos importantes

Para probar esta relación lo más simple es hacer un scatter plot, revisar si existe linealidad y ver si hay valores extremos.

```python
# Pintamos el modelo lineal
sns.regplot(x="drug_concentration", y="score", data=dataset)
```
![Regresion](/images/19_1.png)

Como podemos ver existe linealidad y no hay valores extremos importantes. Si hay dudas sobre si es importante o no el valor extremo siempre podemos hacer un análisis con *scipy.stats.zscore*

```python
stats.zscore(dataset)
"""
array([[-1.76514058,  1.67396113],
       [-0.76058903,  0.4708482 ],
       [-0.59874461,  1.00885384],
       [ 0.19931578, -0.73226472],
       [ 0.83553176, -0.25751973],
       [ 0.93040608, -0.99633437],
       [ 1.1592206 , -1.16754436]])
"""
```
Como vemos no existen valores mayores / menores que 3.

## Independencia de errores y multicolinealidad

Me voy a servir de *statsmodels* para obtener algunos de los indicadores que voy a necesitar.

```python
result = sm.OLS( dataset['drug_concentration'].values, dataset['score'].values).fit()
result.summary()
```

<img src="/images/19_2.png" alt="Drawing" style="width: 500px;"/>

Para probar la independencia de errores recurrimos a [Durbin-Watson](https://es.wikipedia.org/wiki/Estad%C3%ADstico_de_Durbin-Watson) el cual nos va a decir si los errores son independientes (H0) o existe correlación entre los errores (Ha).

Con un valor de 0.364 no podemos rechazar la hipótesis nula, es decir, no podemos afirmar que haya una correlación entre los errores.

Para probar que no existe o es leve la [multicolinealidad](https://es.wikipedia.org/wiki/Multicolinealidad) nos hemos de fijar en *Cond. No.* que como vemos es un vajor bajo.

## Homocedasticidad de los valores residuales

Para probar este parámetro lo mejor es pintar los valores residuales y ver que no tienen ningún patrón.

```python
sns.residplot(x="drug_concentration", y="score", data=dataset)
```

![Regresion](/images/19_3.png)

Podemos apreciar que no hay ningún patrón.

## Los [valores residuales](https://en.wikipedia.org/wiki/Errors_and_residuals) se distribuyen normalmente.

```python
# Calculamos los valores residuales
predictions = regr.predict(np.vstack(dataset['drug_concentration'].values))
residuals = (np.vstack(dataset['score'].values)) - predictions

# Pasamos el test de shapiro para revisar normalidad.
stats.shapiro(np.ravel(residuals))
# (0.9401566982269287, 0.6401315927505493)
```

No podemos rechazar la hipótesis nula sobre la normalidad de los datos.

## Regresión lineal.

Una vez comprobadas las hipótesis vamos a realizar el test.


```python
# Obtenemos los distintos conjuntos de datos
trainset, testset, trainsol, testsol = train_test_split(dataset['drug_concentration'].values,
                                                        dataset['score'].values, test_size=0.33, random_state=42)

# Construimos el modelo con los datos de entrenamiento
regr = linear_model.LinearRegression()
regr.fit(np.vstack(trainset), np.vstack(trainsol))

# Realizamos la valoración de nuestro modelo sobre los datos de prueba
regr.score(np.vstack(testset), np.vstack(testsol))
# 0.91863291159556715

# Realizamos una predicción
regr.predict([[1]])
# array([[ 80.11440735]])
```
