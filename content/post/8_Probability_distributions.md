+++
date = "2016-08-01T19:08:11+02:00"
draft = true
title = "Análisis de datos - Distribuciones de probabilidad <Python>"
description = "Algunas distribuciones de probabilidad"
slug = "probability-distributions"
tags = ['Estadistica', 'python']
+++

Una distribución de probabilidad describe como está distribuida una variable dentro de su rango de valores y la probabilidad de encontrar determinados valores. Existen actualmente [multitud](http://docs.scipy.org/doc/scipy/reference/stats.html) de distribuciones que podemos utilizar para mapear nuestros datos, voy a poner aquí las más comunes.

## Distribución normal

La distribución normale es una distribución de probabilidad continua caracterizada por su centro (media) y desviación típica donde la mayor parte de los valores están cercanos a la media. Aproximadamente cerca del 68% de los datos están dentro del intervalo de una desviación típca y el 95 dentro de dos desviaciones típicas.

Es probablemente la distribución más importante, y refleja con exáctitud multitud de fenomenos del mundo real. Es además precondición de muchos test estadísticos.

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import scipy.stats  as stats

x_axis = np.arange(-3, 3, 0.001)
plt.plot(x_axis, stats.norm.pdf(x_axis,0,1))
```
![Uniform](/images/9_2.png)

## ¿Qué nos aporta una distribución de probabilidad?

Una vez que hemos ajustado nuestros datos a una distribución con sus parámetros deficidos (media, desviación típida, ... ) ya podemos utilizar las funciones que nos proporciona para hacer nuestras estimaciones y cálculos, en particular si tenemos una distribución con media 100 y desviación típica de 20:


* Probability density function (PDF): Nos permite calcular la probabilidad de encontrar un determinado suceso, por ejemplo ¿Qué probabilidad hay de encontrar 125?"

```pyt
stats.norm(100,20).pdf(125)
# 0.009132454269451095
```

* Cumulative distribution function (CDF): Nos da la probabilidad de obtener un valor menor al indicado, por ejemplo "Si cogemos un día al azar, ¿Qué probabilidad hay de que se den al menos 3718 horas de uso?"

```pyt
stats.norm(100,20).cdf(125)
# 0.89435022633314465
```

* Confidence interval : Nos da un intervalo de confianza para un determinado porcentaje, por ejemplo "¿Cúal es el intervalo de valores que puedo obtener con un 63% de probabilidad?"

```pyt
stats.norm.interval(0.63, 100, 20)
# (82.070532719961676, 117.92946728003832)
```

* Survival function (SF): 1-CDF : Nos da la probabilidad de obtener un valor mayor al indicado, por ejemplo "Si cogemos un suceso al azar, ¿Qué probabilidad hay de que se den más de 88?"

```pyt
stats.norm(100,20).sf(88)
# 0.72574688224992645
```

* Percentile point function (PPF): Es la inversa de CDF y nos responde a la pregunta, ¿Dada una cierta probabilidad, cual es el valor del CDF?

```pyt
stats.norm.ppf(0.63, 100, 20)
# 106.63706692873633
```

* Inverse survival function (ISF): Es la inversa de SF, que de igual modo responde a la pregunta, ¿Dada una cierta probabilidad, cual es el valor de la SF?

```pyt
stats.norm.isf(0.63, 100, 20)
# 93.362933071263669
```

* Random Variables: Nos permite simular un conjunto de sucesos en el tiempo dentro de la distatsribución.

```pyt
stats.norm.rvs(100,20,10)
# array([  94.25048457,   85.35982312,   88.16113985,   86.9929707 ,
#          90.98207244,  136.12808423,   88.9124151 ,  106.61102245,
#          83.20154653,   87.28349134])
```



## Distribución uniforme

Se da cuando todos los valores tienen la misma probabilidad de presentarse, y los valores fuera de rango simplemente no ocurren.

```python
%matplotlib inline

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import scipy.stats  as stats

udistro = stats.uniform.rvs(1000,0,5)
pd.DataFrame(udistro).plot(xlim=(-5,10))
```

![Uniform](/images/9_1.png)


## Distribución Binomial


La distribución binomial es una distribución de probabilidad discreta que modela el resultado de un experimento. Se caracteriza por la probabilidad de obtener un determinado suceso y el numero de sucesos, obteniendo la probabilidad de obtener un determinado resultado en un numero de suscesos.

El ejemplo típico es el lanzamiento de una moneda 10 veces, con una probabilidad de suceso del 50%.

```python
# Lanzamiento de 10 monedas 1000 veces
coins = stats.binom.rvs(n=10, p=0.5, size=1000)
pd.DataFrame(coins).hist(range=(-0.5,10.5), bins=11)
```

![Binomial](/images/9_3.png)

Ahora podemos calcular por ejemplo, ¿Cuál es la probabilidad de obtener 9 sucesos?

```python
1- stats.binom.cdf(k=9, n=10, p=0.5)
# 0.0009765625
```


## Distribuciones Geométrica - Exponencial

Las distribuciones geométrica y exponencial modelan el tiempo que requiere un evento para ocurrir:

* La distribución geométrica (discreta) es discreta y modela el número de intentos que toma lograr un suceso en repetidos experimentos partiendo de una probabilidad de ocurrencia.
* La distribución exponencial (continua) modela el tiempo que toma que un evento ocurra partiendo de un ratio de ocurrencia temporal.


Comenzamos con la distribución geométrica, modelando el lanzamiento de monedas. ¿Cuantos intentos debemos lanzar para obtener una cara?

```python
wait_for_head = stats.geom.rvs(size=10000, p=0.5)
print( pd.crosstab(index="counts", columns= wait_for_head))
# col_0     1     2     3    4    5    6   7   8   9   10  11  12  13  15  19
# counts  4981  2513  1202  626  330  180  89  43  13  12   1   5   3   1   1

pd.DataFrame(wait_for_head).hist()
```

La interpretacion es que ha habido 4981 ocasiones en las que ha bastado 1 intento, 2513 ocasiones en las que hemos necesitado 2 intentos y así sucesivamente hasta en una ocasión en la que hemos necesitado 19 intentos.

![Geométrica](/images/9_4.png)

Ahora podemos utilizar las funciones antes mencionadas, por ejemplo. ¿Cuál es la probabilidad de obtener una cara en los primeros 5 intentos?

```python
stats.geom.cdf(k=5, p=0.5)
# 0.96875
```

O la probabilidad de exáctamente obtener cara en el tercer intento.

```python
stats.geom.pmf(k=2,p=0.5)
# 0.25
```

Ahora vamos a utilizar la distribución exponencial.

```python
time_for_head = stats.geom.rvs(size=10000, p=0.5)
plt.fill_between(x=np.arange(0,8,0.01),
                 y1= stats.expon.pdf(np.arange(0,8,0.01)) ,
                 facecolor='green',
                 alpha=0.35)
```

![Geométrica](/images/9_5.png)

Para ver la probabilidad de esperar más de un periodo de tiempo (previamente definido) para que un suceso ocurra.
```python
prob_wait = stats.expon.cdf(x=1, scale=1)
1 - prob_wait
# 0.36787944117144233
```

## Distribución de Poisson

La distribución de Poisson modela la probabilidad de ocurrencia de un determinado número de sucesos en un intervalo de tiempo, teniendo en cuenta que el tiempo que tarda el próximo suceso en ocurrir está modelado por una distribución exponencial.

Esta distribución es muy útil y puede encajar en problemas como predecir el número de emails a recibir en una semana, problemas de tráfico, etc. En los ejemplos vamos a hablar de minutos, aunque la unidad de tiempo podría ser cualquiera a determinar a priori.

Vamos a modelar una distribución que refleja "recibo un email por minuto"

```python
received_mail = stats.poisson.rvs(size=10000, mu=1 )
print( pd.crosstab(index="counts", columns= received_mail))

# col_0      0     1     2    3    4   5  6                                    
# counts  3694  3620  1850  640  155  35  6
```

Según este cuadro muestra la frecuencia de llegada y lo extraño que es que lleguen más de un correo en cada minuto, por ejemplo sólo en 6 minutos de los 10000 evaluados se han recibido 6 correos.

Por ejemplo, vamos a calcular la probabilidad de recibir 5 correos en un minuto.

```python
1- stats.poisson.cdf(k=5, mu=1)
# 0.00059418481758166664
```
