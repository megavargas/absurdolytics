+++
date = "2016-07-03T19:08:11+02:00"
draft = true
title = "Aprendizaje automático - Naive Bayes "
description = "Posibilidades, funcionamiento y ejemplos de uso del algorimo Naive Bayes para clasificación y aprendizaje automáatico"
slug = "machine-learning-naive-bayes"
tags = ['Aprendizaje automático', 'Clasificación', 'sikit-learn', 'mllib', 'spark']
+++

## Introducción

El algoritmo de aprendizaje automático Naive Bayes es una técnica de *clasificación* basada en el [Teorema de Bayes](https://es.wikipedia.org/wiki/Teorema_de_Bayes) que asume independiencia entre las muestras.

Este modelo es relativamente sencillo de construir y particularmente útil cuando tenemos que enfrentarnos a grandes conjuntos de datos, logrando unos resultados que superan en muchas ocasiones a algoritmos más complejos.

En contraprestación, este algoritmo asume sucesos totalmente independiente, lo que es realmente complicado de obtener en la vida real.

## Aplicación de este algoritmo

Se trata de un algoritmo de *clasificación* y por lo tanto puede ser utilizado para este tipo de problemas con ciertas peculiaridades inerentes a su algoritmo:

* Clasificación de textos
* Clasificación de múltiples clases resultados
* Sistemas de recomendación

También, y dada su simplicidad y rapidez puede ser utilizado en muchos casos en tiempo real, realizando clasificaciones y predicciones de forma rápida y fiable.

## Ejemplo de clasificación con scikit learn

Scikit learn es una potente y versatil herramienta que nos permite realizar aprendizaje automático, mineria de datos y análisis de datos. Está basada en Numpy, SciPy y matplotlib y es software libre.

Adjunto un ejemplo donde descargamos un conjunto de datos para la verificación de cheques bancarios a través de las características de sus imágenes.

```pyt

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from sklearn.naive_bayes import GaussianNB
from sklearn.cross_validation import train_test_split

# Data obtained from archive.ics.uci.edu
dataset = pd.read_csv('https://archive.ics.uci.edu/ml/machine-learning-databases/00267/data_banknote_authentication.txt',
                     names=['variance', 'skewness', 'curtosis','entropy', 'result'])

x = np.array(dataset.loc[:,['variance', 'skewness', 'curtosis','entropy']])
y = np.array(dataset.loc[:,'result'])

trainset, testset, trainsol, testsol = train_test_split(x, y, test_size=0.33, random_state=42)

clf = GaussianNB()
clf.fit(trainset, trainsol)

clf.score(testset, testsol)
# 0.81159420289855078

clf.predict([-3.380000,-0.707700,2.532500,0.718080])
# array([1])
# True banknote

clf.predict([1,1,1,1])
array([0])
# False banknote
```

## Ejemplo de clasificación con spark.mllib

[MLLib](http://spark.apache.org/mllib/) es una herramienta que proporciona varias utilidades que ayudan en las tareas de aprendizaje automático y análisis de datos, incluye utilidades que son adecuadas para clasificación, regresión, agrupación, etc.

Esta herramienta está integrada con Spark y puede utilizarse por tanto dentro del stack de hadoop, vamos a ver un ejemplo de uso con unos datos sobre [diabetes](https://archive.ics.uci.edu/ml/datasets/Pima+Indians+Diabetes).

```pyt

from pyspark.mllib.classification import NaiveBayes, NaiveBayesModel
from pyspark.mllib.regression import LabeledPoint

def parseLine(line):
    play = line.split(',')
    label = float(play[-1])
    features = [float(x) for x in play[0:-1]]
    return LabeledPoint(label,features)

# tttdata = "https://archive.ics.uci.edu/ml/machine-learning-databases/pima-indians-diabetes/pima-indians-diabetes.data"
data = sc.textFile('/tmp/tttdata.csv').map(parseLine)

'''
1. Number of times pregnant
2. Plasma glucose concentration a 2 hours in an oral glucose tolerance test
3. Diastolic blood pressure (mm Hg)
4. Triceps skin fold thickness (mm)
5. 2-Hour serum insulin (mu U/ml)
6. Body mass index (weight in kg/(height in m)^2)
7. Diabetes pedigree function
8. Age (years)
9. Class variable (0 or 1)
'''

training, test = data.randomSplit([0.6,0.4],seed=0)
model = NaiveBayes.train(training,1.0)
predictionAndLabel = test.map(lambda p: (model.predict(p.features),p.label))
accuracy = 1.0 * predictionAndLabel.filter(lambda (x, v): x == v).count() / test.count()

print(accuracy)
# 0.601973684211

print(model.predict([3,158,76,36,245,31.6,0.851,28]))
# 1.0
```
