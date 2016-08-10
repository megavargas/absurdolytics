+++
date = "2016-08-07T20:08:11+02:00"
draft = true
title = "Análisis de datos - ANOVA <Python>"
description = "Dónde aplicar, cómo ejecutar y cómo interpretar test de ANOVA"
slug = "anova-manova"
tags = ['Estadistica', 'python', 'ANOVA','MANOVA']
+++

La herramienta ANOVA se utiliza en el nálisis de la media de varios grupos de control (variable independiente) con respecto a una variable dependiente y continua.

Para el uso de esta herramienta debemos partir de las siguientes precondiciones:

* La variable dependiente está normalmente distribuida en todos los grupos
* No hay valores extremos significativos en ningún grupos
* Las varianzas de los grupos son similares
* Factores independientes

## Un factor (one way ANOVA)

Para realizar este análisis vamos a construir un dataset de datos normalizados que simulan un experimento del tipo tratamiento->resultado.

``` python
import pandas as pd
import numpy as np
from scipy import stats
import matplotlib.pyplot as plt

placebo = pd.DataFrame(stats.norm.rvs(loc=5,scale=1,size=500))
lowdose = pd.DataFrame(stats.norm.rvs(loc=7,scale=1,size=500))
highdose = pd.DataFrame(stats.norm.rvs(loc=10,scale=1,size=500))
```

Obviamente las variables serán normales y cumplirán las condiciones de extremos y varianzas similares, no obstante os dejo el código para comprobarlo.

```python
# Test normalidad
stats.shapiro(placebo)
stats.shapiro(lowdose)
stats.shapiro(highdose)

# Test varianzas de los grupos similares
stats.levene(placebo, lowdose, highdose, center='mean')
# LeveneResult(statistic=array([ 1.35371227]), pvalue=array([ 0.25859566]))
# Aceptamos hipótesis nula, son similares
```

Ahora ejecutamos el test ANOVA para los distintos grupos.

```python
stats.f_oneway(placebo, lowdose, highdose)
# F_onewayResult(statistic=array([ 2966.71214116]), pvalue=array([ 0.]))
```
Podemos rechazar la hipótesis nula (p-value < 0.05) y concluir que los diferentes tratamientos tienen diferentes resultados.

Ahora necesitamos analizar en detalle la diferencia entre cada tratamiento. Podríamos intentar realizar un t-test a cada pareja, no obstante podríamos obtener resultados erróneos si mantenemos el nivel de significancia al 5%. Una opción podría ser dividir este nivel de significancia entre el número de posibles estudios ([Bonferroni Correction](https://en.wikipedia.org/wiki/Bonferroni_correction)) pero aún así podríamos rechazar algún resutado que fuera significativo.

En estos casos podemos utilizar un test de [Turkey HSD](https://en.wikipedia.org/wiki/Tukey%27s_range_test) o [Scheffe](https://en.wikipedia.org/wiki/Scheff%C3%A9%27s_method). Os dejo aquí como utilizar el primero.

```python

# Concateno los datasets para formar un único datasets
placebo['dose']='placebo'
lowdose['dose']='lowdose'
highdose['dose']='highdose'

dataset = pd.concat([placebo, lowdose, highdose], axis=0)
dataset.columns = ['result','dose']

from statsmodels.stats.multicomp import pairwise_tukeyhsd

tukey = pairwise_tukeyhsd(endog=dataset['result'], groups=dataset['dose'], alpha=0.05)
tukey.summary()

"""
Multiple Comparison of Means - Tukey HSD,FWER=0.05
group1	group2	meandiff	lower	upper	reject
highdose	lowdose	-2.906	-3.0578	-2.7543	True
highdose	placebo	-4.9582	-5.11	-4.8065	True
lowdose	placebo	-2.0522	-2.2039	-1.9004	True
"""

%matplotlib inline

tukey.plot_simultaneous()    # Plot group confidence intervals
plt.vlines(x=49.57,ymin=-0.5,ymax=4.5, color="red")
```

Podemos ver que la difernecia es entre todas los grupos y los intervalos de confianza.

![Turkey](/images/15_1.png)

## Múltiples factores (Two/Three ways ANOVA)

En este ejemplo vamos a utilizar un dataset sobre el sabor de las patatas según donde han crecido, tiempo en almacen, método de cocinado etc. con respecto a su savor.

```python

import pandas as pd
import numpy as np
from scipy import stats
import matplotlib.pyplot as plt

# Vamos a utilizar statsmodels ya que nos ofrece una formula sencilla para realizar análisis con múltiples factores
from statsmodels.formula.api import ols
from statsmodels.stats.anova import anova_lm

dataset = pd.read_csv('http://www.stat.ufl.edu/~winner/data/potato.dat',delim_whitespace=True, usecols=[1,3,4,5,6,7],header=None)
dataset.columns = ['growing_area','size_of_potato','storage','cooking','texture','flavour']

# Vamos a analizar primero dos factores
formula = 'texture ~ C(storage) + C(cooking) + C(storage):C(cooking)'
lm = ols(formula, dataset).fit()
print(anova_lm(lm))

"""
df    sum_sq   mean_sq         F    PR(>F)
C(storage)              4.0  1.343500  0.335875  4.923695  0.001325
C(cooking)             23.0  2.067186  0.089878  1.317544  0.183137
C(storage):C(cooking)  92.0  5.990322  0.065112  0.954500  0.587128
Residual               81.0  5.525500  0.068216       NaN       NaN
"""
```

La formula sigue un formato conocido si habéis trabajado con *R* y establece flavour como variable dependiente, dos factores aislados y la combinación de los dos factores anteriores.

Podemos concluir que el factor almacenamiento tiene impacto en la variable *sabor* mientras que no podemos concluir que el método de cocinado o la combinación de almacenamiento / cocinado tenga impactoself.

Ahora vamos a ver un ejemplo para el análisis de tres factores:

```python

formula = "texture ~ C(storage) + C(cooking) + C(growing_area) + C(storage):C(cooking) \
                                + C(storage):C(growing_area) + C(cooking):C(growing_area)"
lm = ols(formula, dataset).fit()
print(anova_lm(lm))

"""
df    sum_sq   mean_sq          F    PR(>F)
C(storage)                   4.0  1.343500  0.335875   5.246344  0.001019
C(cooking)                  23.0  2.067186  0.089878   1.403882  0.144723
C(growing_area)              1.0  1.633884  1.633884  25.521158  0.000004
C(storage):C(cooking)       92.0  4.641388  0.050450   0.788023  0.853492
C(storage):C(growing_area)   4.0  0.058160  0.014540   0.227113  0.922249
C(cooking):C(growing_area)  23.0  1.743902  0.075822   1.184331  0.291349
Residual                    64.0  4.097330  0.064021        NaN       NaN
"""

```
Los resultados son similares, añadiendo *origen* como factor con impacto en el saborself.

# Múltiple variables dependientes (MANOVA)

En el caso en que queramos analizar múltiples variables dependientes (MANOVA) deberíamos repetir estos tests modificando la formula, ya que no existe ningún método directo que podamos utilizar a no ser que recurramos a [rpy2](http://rpy2.bitbucket.org/).

Por ejemplo, para calcular el efecto sobre el sabor:

```python

formula = "flavour ~ C(storage) + C(cooking) + C(growing_area) + C(storage):C(cooking) \
                                + C(storage):C(growing_area) + C(cooking):C(growing_area)"
lm = ols(formula, dataset).fit()
print(anova_lm(lm))

"""
df     sum_sq   mean_sq          F        PR(>F)
C(storage)                   4.0   2.820875  0.705219  13.761994  3.743047e-08
C(cooking)                  23.0  19.037607  0.827722  16.152584  1.308596e-18
C(growing_area)              1.0   0.179279  0.179279   3.498537  6.599714e-02
C(storage):C(cooking)       92.0   3.947288  0.042905   0.837276  7.841929e-01
C(storage):C(growing_area)   4.0   0.451306  0.112827   2.201753  7.867201e-02
C(cooking):C(growing_area)  23.0   1.718494  0.074717   1.458067  1.201155e-01
Residual                    64.0   3.279612  0.051244        NaN           NaN
"""
