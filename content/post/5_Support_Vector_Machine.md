+++
date = "2016-07-05T19:08:11+02:00"
draft = true
title = "Aprendizaje automático - Clasificación - Support Vector Machine Learning"
description = "Ejemplos de uso de SVM con scikit-learn y mllib"
slug = "machine-learning-SVN"
tags = ['Aprendizaje automático', 'Clasificación', 'scikit-learn', 'mllib', 'spark']
+++

## Introducción

El algoritmo de aprendizaje automático SVM es una técnica **no probabilistica** de **clasificación lineal supervisada** que nos permit construye un modelo lineal con el que predecir la clase de los nuevos sucesos.

Lo que realmente intenta este algoritmo es encontrar un hiperplano que dejaría la mayor distancia entre los sucesos de distinta clase, para asi clasificar y predecir futuros sucesos.

Además de permitir una clasificación lineal, esta herramienta puede realizar clasificaciones no lineales, usando una técnica usualmente llamada *kernel trick*, que implica mapear los inputs en una dimensión mayor.

## Configuración de SVN

Para configurar un algoritmo SVM vamos a tener que introducir algunas variables, que modificarán por completo la eficiencia y acierto de nuestor algoritmo:

*  Kernel trick - Técnica para permitir clasificaciones no lineales, algunas de las más conocidas que vamos a encontrar son la función [polinomial](https://en.wikipedia.org/wiki/Polynomial_kernel), la función [radial](https://en.wikipedia.org/wiki/Radial_basis_function_kernel) y la función [sigmoide](https://en.wikipedia.org/wiki/Sigmoid_function)
* C - Parámetro que nos permite modificar el coste de "clasificar mal" y el margen de error de nuestro hiperplano. Cabría pensar que cuanto mayor mejor, pero podemos caer en problemas por *overfitting* y es mejor probar con muestras y ver cuál es la mejor opción.
* Gamma - Es un parámetro de la función de kernel, que nos va a permitir ajustar el algorimpo para permitir mayor o menor varianza. Al igual que el anterior cabría pensar que cuanto menor varianza mejor, pero siempre es mejor ajustar para no tener problemas de recursos y *overfitting*.

Si queréis algún método automático que busque los mejores parámetros podéis mirar en [aquí](https://www.csie.ntu.edu.tw/~cjlin/libsvm/index.html)

## Ejemplo con scikit-learn

Scikit learn es una potente y versatil herramienta que nos permite realizar aprendizaje automático, mineria de datos y análisis de datos. Está basada en Numpy, SciPy y matplotlib y es software libre.

Adjunto un ejemplo donde descargamos un conjunto de datos para la verificación de cheques bancarios a través de las características de sus imágenes.

```pyt
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from sklearn.cross_validation import train_test_split
from sklearn import svm

dataset = pd.read_csv('https://archive.ics.uci.edu/ml/machine-learning-databases/00267/data_banknote_authentication.txt',
                     names=['variance', 'skewness', 'curtosis','entropy', 'result'])

X = np.array(dataset.loc[:,['variance', 'skewness', 'curtosis','entropy']])
y = np.array(dataset.loc[:,'result'])

trainset, testset, trainsol, testsol = train_test_split(X, y, test_size=0.33, random_state=42)

clf = svm.SVC()
clf.fit(trainset, trainsol)

# SVC(C=1.0, cache_size=200, class_weight=None, coef0=0.0,
#  decision_function_shape=None, degree=3, gamma='auto', kernel='rbf',
#  max_iter=-1, probability=False, random_state=None, shrinking=True,
#  tol=0.001, verbose=False)

clf.score(testset, testsol)
# 1.0

clf.predict([-3.380000,-0.707700,2.532500,0.718080])
# array([1])

clf.predict([1,1,1,1])
# array([0])

```

## Ejemplo con Spark

Ahora vamos a cambiar a realizar un ejemplo similar con *scala* y *spark* tomando los datos de [diabetes](https://archive.ics.uci.edu/ml/datasets/Pima+Indians+Diabetes) de [uci](https://archive.ics.uci.edu/) para evaluar nuestro algoritmo.

```sca


import org.apache.spark.mllib.classification.{SVMModel, SVMWithSGD}
import org.apache.spark.mllib.evaluation.BinaryClassificationMetrics
import org.apache.spark.mllib.util.MLUtils
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.mllib.regression.LabeledPoint

val data = sc.textFile("/tmp/diabetes.csv")

val parsedData = data.map { line =>
  val parts = line.split(',')
  LabeledPoint(parts(8).toDouble, Vectors.dense(parts.slice(0,7).map(_.toDouble)))
}

// Separamos los datos para training (60%) y test (40%).
val splits = parsedData.randomSplit(Array(0.6, 0.4), seed = 11L)
val training = splits(0)
val test = splits(1)

// Ejecutamos el algoritmo
val numIterations = 100
val model = SVMWithSGD.train(training, numIterations)

// Calculamos los valores para nuestro conjunto de test
val scoreAndLabels = test.map { point =>
  val score = model.predict(point.features)
  (score, point.label)
}

// Obtenemos las métricas
val metrics = new BinaryClassificationMetrics(scoreAndLabels)
val auROC = metrics.areaUnderROC()

println("Area under ROC = " + auROC)
```
