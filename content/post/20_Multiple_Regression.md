+++
date = "2016-08-10T21:08:11+02:00"
draft = true
title = "Análisis de datos - Regresión Múltiple <Python>"
description = "Ejemplo de Regresión Múltiple con python y scikit-learn"
slug = "Multiple-Regression"
tags = ['Estadistica', 'python', 'Multiple-Regression']
+++

Este método, al igual que los métodos para evaluar la correlación entre variables mide la relación entre una variable dependiente y una variable independiente, en cambio aquí no vamos a obtener un valor de correlación sino un modelo a través del cual podemos predecir nuevos valores.

La regresión lineal es el método más simple de regresión. En este post vamos a utilizar el método de [mínimos cuadrados](https://es.wikipedia.org/wiki/M%C3%ADnimos_cuadrados) que intenta encontrar la función *lineal* que mejor se aproxima los datos de acuerdo con el criterio de mínimo error cuadrático.

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

# Creamos el modelo
regr = linear_model.LinearRegression()
regr.fit(dataset[['drug_concentration']], dataset[['score']])

# Hacemos una predicción
regr.predict([[1]])
# array([[ 80.11440735]])

# Pintamos el modelo lineal
sns.regplot(x="drug_concentration", y="score", data=dataset)
```

![Regresion](/images/19_1.png)


Este sería el uso más simple de este algoritmo, no obstante usualmente reservamos parte de nuestros datos para probar el mismo modelo y evaluar la exáctitud.

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
```

La exáctitud de nuestro modelo es bastante buena.
