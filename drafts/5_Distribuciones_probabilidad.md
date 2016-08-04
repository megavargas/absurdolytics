+++
date = "2016-07-01T19:08:11+02:00"
draft = true
title = "Indicadores estadísticos básicos"
description = "¿Qué es una distribución de probabilidad? ¿Cuál uso? ¿Qué puedo obtener? Algún ejemplo"
slug = "distribuciones-probabilidad-scipy"
tags = ['estadística']
+++

Muchas veces al enfrentarnos con problemas donde tenemos que realizar ciertas inferencias, valorar hipótesis, establecer intervalos de confianza, etc sobre un conjunto de datos y no conocemos con detalle la función de probabilidad que rige nuestros sucesos **debemos acudir las distribuciones de probabilidad**.

Una distribución de probabilidad no es más que una **función conocida** que asigna a cada suceso sobre nuestras variables la probabilidad de que dicho suceso ocurra. Estas distribuciones nos van a permitir aproximar y realizar estimaciones con facilidad.

Este post va a desgranar algunas de las más utilizadas con algunos ejemplos prácticos con python y librerías estadísticas como pandas, scipy, etc.

## Estudio de tus datos

Antes de meternos en las distribuciones más comunes y en intentar ver qué distribución podemos utilizar debemos intentar echar un vistazo a nuestros datos.

Primero vamos a intentar obtener los parámetros más comunes, como la media, el error estándar, la mediana, la diferencia entre máximo y mínimo y hacer algunas gráficas para ver como son nuestros datos.

Test con un random de 500 números.

```pyt
import pandas as pd
import scipy.stats as stats

data = {'sample':randn(500)}
df = pd.DataFrame(data)
df.describe()

plot(df,'.')
hist(df['sample'])
boxplot(df['sample'])

'''
count	500.000000
mean	-0.002110
std	0.998561
min	-3.204273
25%	-0.697430
50%	0.005242
75%	0.698710
max	2.494859
'''

```

||||
|---|---|---|
|![scatter](/images/5_1.png)|![scatter](/images/5_2.png)|![scatter](/images/5_3.png)|

Como veremos a continuación, a tenor de los datos y las gráficas podríamos utilizar una distribución normal para aproximar y hacer algunas estimaciones e inferencias. En cualquier caso este paso siempre es necesario previamente al uso de distribuciones de probabilidad.


## Distribuciones de probabilidad ¿Cómo se cuál utilizar?

Como comentamos, el primer paso para intentar utilizar una distribución con nuestros datos es hacer un análisis preliminar, pintar los datos y analizar que vemos.

Para profundizar existen multitud de librerías que nos van a permitir ajustar las distintas distribuciones a nuestros datos y ver cómo encajan, aquí vamos a mostrar cómo hacerlo con [scipy](https://www.scipy.org/).

Existen 89 distribuciones en [scipy](https://www.scipy.org/) con las cuales podemos intentar aproximar nuestros datos, para esto existe una función *fit* que nos da los parámetros de la distribución a aproximar con nuestros datos. Con estos parámetros podemos comparar amabas distribuciones y ver si efectivamente podemos utilizarla.

Por ejemplo vamos a traer los datos de uso de bicicletas diarios de [BiciMad](http://www.bicimad.com/) que hemos obtenido de la plataforma de datos abiertos de madrid. A partir de estos datos vamos a:

* Obtener el histograma de los datos reales
* Obtener los parámetros simulando una distribución normal de los datos
* Comparar ambas distribuciones

Os dejo aquí el código.

```pyt
import numpy as np
import pandas as pd
import scipy.stats as st
import scipy
import matplotlib.pyplot as plt
import seaborn as sns

data = pd.read_csv('http://datos.madrid.es/egob/catalogo/216343-0-bicimad-disponibilidad.csv',
                   sep=';', thousands='.', decimal=',').dropna()

service_hours_data = data['HORAS_TOTALES_USOS_BICICLETAS']

service_hours_data.describe()
# count     322.000000
# mean     3718.672329
# std      1265.696881
# min       759.880000
# 25%      2877.845000
# 50%      3476.075000
# 75%      4656.557500
# max      6680.650000

params = st.norm.fit(service_hours_data)
# (3718.6723291925464, 1263.7299846706792)

x = scipy.linspace(0,7000,20)

pdf_fitted= st.norm.pdf(x,loc=params[0],scale=params[1])
plt.hist(service_hours_data,normed=1)
plt.plot(x,pdf_fitted)

plt.show()

```

![comparación](/images/5_4.png)

Como podemos ver nos encaja bastante con una distribución normal **N(3718,1263)**, no obstante quizá esté un poco cargada hacia la izquierda, podríamos probar con una distribución [Raileigh](https://en.wikipedia.org/wiki/Rayleigh_distribution).

A partir de este código puedes probar otras distribuciones hasta que se ajuste. También podrías automatizar esta tarea recorriendo las distribuciones que nos aporta *scipy* y comprobar automáticamente las diferencias, os dejo un par de cuestiones de stackoverflow por donde podéis empezar:

* [fitting-empirical-distribution-to-theoretical-ones-with-scipy-python](http://stackoverflow.com/questions/6620471/fitting-empirical-distribution-to-theoretical-ones-with-scipy-python)
* [fitting-distributions-goodness-of-fit-p-value-is-it-possible-to-do-this-with](http://stackoverflow.com/questions/6615489/fitting-distributions-goodness-of-fit-p-value-is-it-possible-to-do-this-with/16651524#16651524)

## ¿Qué nos aporta una distribución de probabilidad?

Una vez que hemos ajustado nuestros datos a una distribución con sus parámetros deficidos (media, desviación típida, ... ) ya podemos utilizar las funciones que nos proporciona para hacer nuestras estimaciones y cálculos, en particular:

* Probability density function (PDF): Nos permite calcular la probabilidad de encontrar un determinado suceso, por ejemplo "Si cojemos un día al azar, ¿Qué probabilidad hay de que se den exáctamente 3718 horas de uso?"

```pyt
st.norm(3718,1263).pdf(3718)
#0.00031586878891641545
```

* Cumulative distribution function (CDF): Nos da la probabilidad de obtener un valor menor al indicado, por ejemplo "Si cogemos un día al azar, ¿Qué probabilidad hay de que se den al menos 3718 horas de uso?"

```pyt
st.norm(3718,1263).cdf(3718)
#0.5
```

* Confidence interval : Nos da un intervalo de confianza para un determinado porcentaje, por ejemplo "¿Cúal es el intervalo de valores que puedo obtener con un 63% de probabilidad?"

```pyt
st.norm.interval(0.63, 3718, 1263)
#(2585.7541412655801, 4850.2458587344199)
```

* Survival function (SF): 1-CDF : Nos da la probabilidad de obtener un valor mayor al indicado, por ejemplo "Si cogemos un día al azar, ¿Qué probabilidad hay de que se den más de 3718 horas de uso?"

```pyt
st.norm(3718,1263).sf(4718)
#0.21424867105266132
```

* Percentile point function (PPF): Es la inversa de CDF y nos responde a la pregunta, ¿Dada una cierta probabilidad, cual es el valor del CDF?

```pyt
st.norm.ppf(0.63, 3718, 1263)
# 4137.1307765496995
```

* Inverse survival function (ISF): Es la inversa de SF, que de igual modo responde a la pregunta, ¿Dada una cierta probabilidad, cual es el valor de la SF?

```pyt
st.norm.isf(0.63, 3718, 1263)
3298.8692234503005
```

* Random Variables: Nos permite simular un conjunto de sucesos en el tiempo dentro de la distribución.

```pyt
st.norm.rvs(3718,1263,10)
# array([ 4590.99169745,  5243.88240355,  3924.41552242,  2224.41764711,
#         4866.65056302,  4061.75940057,  2289.55149949,  4819.11184994,
#         3527.26788658,  3573.42324586])
```
