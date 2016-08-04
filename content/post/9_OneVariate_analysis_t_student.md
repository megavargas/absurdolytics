+++
date = "2016-08-01T19:08:11+02:00"
draft = true
title = "Análisis de una variable"
description = "Análisis de una variable con la distribución t-student, binomial y Chi"
slug = "onevariate-t-student-clt"
tags = ['Estadistica', 'python', 't-student', 'binomial','chi-cuadrado']
+++


## t-student

La distribución t de Student se suele utilizar para **hacer estimaciones de la media** cuando se desconoce la varianza (es lo habitual) y se usan muestas pequeñas.

Podríamos analizar por ejemplo si una muestra obtenida de una población tiene las mismas características que la población, o evaluar la diferencia entre dos muestras.

Para ser utilizada se deben dar las siguientes condiciones:

* La variable a estudiar está normalmente distribuida
* La variable a estudiar no presenta muchos valores extremos

Para este ejemplo vamos a partir de un conjunto de datos del portal de datos abiertos de Madrid sobre el uso de BiciMad, concretamente sobre el parámetro HORAS_TOTALES_USOS_BICICLETAS.

Como hemos visto en el post anterior, no estamos ante una distribución normal, no obstante y gracias al teorema central del límite podemos tratar la media de las muestras como una distribución normal.

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from scipy import stats

data = pd.read_csv('http://datos.madrid.es/egob/catalogo/216343-0-bicimad-disponibilidad.csv', sep=';', thousands='.', decimal=',').dropna()

data['HORAS_TOTALES_USOS_BICICLETAS'].describe()

# count     343.000000
# mean     3708.201574
# std      1231.080804
```

* ¿Qué intervalo medio de uso podemos esperar en 30 días con un intervalo de confianza del 95%?
```python
stats.t.interval(0.95, df = 29, loc = 3688.003445, scale= (1214.821045/np.sqrt(30)))     
# (3234.381811734112, 4141.6250782658881)
```

* ¿Qué probabilidad tenemos de tener una media de 5000 horas en el próximo mes?
```python
stats.ttest_1samp(data['HORAS_TOTALES_USOS_BICICLETAS'], 5000)
# Ttest_1sampResult(statistic=-20.405840879535834, pvalue=7.6100510234310873e-62)
```

## Binomial

Este test se usa cuando queremos evaluar dos condiciones mutuamente excluyentes (cara-cruz, hombre-mujer). Vamos a crear un array arbitrario datos binarios:

```python
pd.DataFrame(np.random.choice(2,1000))[0].value_counts()
# 1    513
# 0    487
```

Ahora probamos con una probabilidad de 0.5 para ambos sucesos y para 0.2 para éxito (1)

```python
stats.binom_test([513,487],p=0.5)
# 0.42920977720933345

stats.binom_test([513,487],p=0.2)
# 6.1837050724813337e-13
```

El test arroja los valores p-value para un intervalo de confianza del 95% (two sided) por lo tanto podemos aceptar la hipótesis nula (la probabilidad de suceso es de 0.5) en el primer caso y rechazar la hipótesis nula (la probabilidad de suceso es de 0.2) en el segundo caso.

## Chi cuadrado

Este test nos sirve para analizar una variable nominal con 3 o más categorías, donde queremos probar una determinada frecuencia con un intervalo de confianza.

Vamos a cargar datos sobre las próximas [actividades culturales de Madrid](http://datos.madrid.es/egob/catalogo/206974-0-agenda-eventos-culturales-100.csv) y contar los distintos tipos de actividades.

```python
import numpy as np
import pandas as pd
from scipy import stats
import matplotlib.pyplot as plot

activities = pd.read_csv('http://datos.madrid.es/egob/catalogo/206974-0-agenda-eventos-culturales-100.csv', sep=';')
frequencies = pd.DataFrame(activities['TIPO'].value_counts())['TIPO']

# /contenido/actividades/CineActividadesAudiovisuales                150
# /contenido/actividades/Fiestas                                      64
# /contenido/actividades/CircoPerformanceTeatro                       24
# /contenido/actividades/Exposiciones                                 21
# /contenido/actividades/ActividadesCalleArteUrbano                   17
# /contenido/actividades/CuentacuentosTiteresMarionetas               15
# /contenido/actividades/Musica                                       13
# /contenido/actividades/CursosTalleres                               11
# /contenido/actividades/ProgramacionDestacadaAgendaCultura            8
# /contenido/actividades/Musica/Flamenco                               6
# /contenido/actividades/ItinerariosOtrasActividadesAmbientales        6
# /contenido/actividades/Musica/DjsElectronicaExperimental             5
# /contenido/actividades/DanzaBaile                                    5
# /contenido/actividades/CineActividadesAudiovisuales/CineFiccion      5
# /contenido/actividades/Musica/JazzSoulFunkySwingReagge               5
# /contenido/actividades/CircoPerformanceTeatro/CircoMagia             4
# /contenido/actividades/ExcursionesItinerariosVisitas                 3
# /contenido/actividades/ConferenciasColoquios                         3
# /contenido/actividades/Musica/RockPop                                2
# /contenido/actividades/CircoPerformanceTeatro/Drama                  2
# /contenido/actividades/Musica/Clasica                                2
# /contenido/actividades/Campamentos                                   2
# /contenido/actividades/Musica/RapHipHop                              2
# /contenido/actividades/ConcursosCertamenes                           2
# /contenido/actividades/CircoPerformanceTeatro/Performance            1
# /contenido/actividades/CircoPerformanceTeatro/ComediaMonologo        1
# /contenido/actividades/DanzaBaile/ClasicaEspanola                    1
# /contenido/actividades/VeranosVilla                                  1
# /contenido/actividades/Musica/Zarzuela                               1
```

Ahora si ejecutamos el test sin parámetros, estaríamos probando que todas las categorías tienen la misma proporción, lo que obviamente no es real.

```python
stats.chisquare(frequencies.values)
# Power_divergenceResult(statistic=1798.0104712041878, pvalue=0.0)
```

Vamos a obtener las verdaderas frecuencias y así obtener un pvalue cercano a 1.

```python
activities['TIPO'].count()
# 382

frequencies_real = pd.DataFrame(activities['TIPO'].value_counts()) / activities['TIPO'].count()
stats.chisquare(frequencies_real.values)
# Power_divergenceResult(statistic=array([ 4.70683369]), pvalue=array([ 0.99999979]))
```
