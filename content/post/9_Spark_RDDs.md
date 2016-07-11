+++
date = "2016-07-08T19:08:11+02:00"
draft = true
title = "Introducción a los RDDs de Spark"
description = "Ejemplos de uso de los RDDs de Spark con python"
slug = "RDDs-Spark-Python"
tags = ['RDDs', 'spark']
+++

Spark gira en torno al concepto de RDDs (Resilient Distributed Datasets), que traducido son colecciones de datos distribuidos donde se aplican técnicas que permiten tener tolerancia a fallos.

Hay básicamente dos formas de crear DDR : *paralelizar* una colección existente o importar datos en un sistema de almacenamiento externo (HDFS , HBase , ...).

* *Paralelizar* una colección

```pyt
data = [x for x in range(1,100000)]
dist_data = sc.parallelize(data)
dist_data.take(5)
```

A partir de este momento podemos actuar sobre *dist_data* en paralelo, por ejemplo:

```pyt
dist_data.map(lambda y : y + 1).reduce(lambda x,y:x+y)
# 5000049999
```

El método *paralellize* realizará automáticamente las particiones por nosotros, teniendo en cuenta nuestra arquitectura. No obstante si tenemos la necesidad podemos especificar el número de particiones mediante un parámetro, por ejemplo ``` paralellize(data,10)```

* Cargando un conjunto de datos externos a través de la función *textFile*

```pyt
data = sc.textFile("/tmp/test.csv")
```

## Operaciones básicas sobre RDDs

Los RDDs soportan dos tipos de operaciones: **Transformaciones**, que crean un nuevo RDD a partir de uno existente, y acciones, que devuelven un valor después de ejecutar un cálculo sobre un RDD.

Por ejemplo, la función *map* es una transformación que aplica a cada elemento del RDD una función y devuelve un nuevo RDD con los resultados. En cambio, la función *reduce* es una acción que aplica a todos los elementos del RDD una función y devuelve el resultado final.

Es importante comentar que todas las transformaciones en Spark son perezosas, y sus resultados sólo se calculan cuando serequiere para alguna acción. Por defecto esta operación de transformación se realiza cada vez que se ejecuta alguna acción, sin embargo podemos hacer que este RDD calculado siga en memoria y así acelerar los trabajos.

## Transformaciones

Algunos ejemplos de transformaciones.

```python

data = ['hola','mundo']
dist_data = sc.parallelize(data)
dist_data.collect()
# ['hola', 'mundo']

dist_data.map(lambda x: x.split(',')).take(1)
# [['hola']]

dist_data.filter(lambda x: x == 'mundo').collect()
# ['mundo']

dist_data.flatMap(lambda x: x.split(',')).take(1)
# ['hola']

dist_data.sample(True,10,5).collect()
# ['hola','hola','hola','hola','hola','hola','hola','mundo','mundo','mundo','mundo','mundo','mundo','mundo','mundo','mundo','mundo']

dist_data.union(dist_data).collect()
# ['hola', 'mundo', 'hola', 'mundo']

dist_data.intersection(sc.parallelize(['mundo'])).collect()
# ['mundo']

dist_data.union(dist_data).distinct().collect()
# ['mundo', 'hola']

dist_data.union(dist_data).map(lambda x: (x,1)).groupByKey().collect()
# [('mundo', <pyspark.resultiterable.ResultIterable at 0x113f38f90>), ('hola', <pyspark.resultiterable.ResultIterable at 0x113d78750>)]

dist_data.union(dist_data).map(lambda x: (x,1)).reduceByKey( lambda x,y: x+y).collect()
# [('mundo', 2), ('hola', 2)]

dist_data.union(dist_data).map(lambda x: (x,1)).aggregateByKey(0,lambda x,y: x+y, lambda x,y:x+y).collect()
# [('mundo', 2), ('hola', 2)]

dist_data.union(dist_data).map(lambda x: (x,1)).sortByKey().collect()
# [('hola', 1), ('hola', 1), ('mundo', 1), ('mundo', 1)]

dist_data.union(dist_data).map(lambda x: (x,1)).join(udist_data.map(lambda x: (x,1))).collect()
# [('mundo', (1, 1)), ('mundo', (1, 1)), ('mundo', (1, 1)), ('mundo', (1, 1)), ('hola', (1, 1)), ('hola', (1, 1)), ('hola', (1, 1)), ('hola', (1, 1))]

dist_data.union(dist_data).map(lambda x: (x,1)).cogroup(udist_data.map(lambda x: (x,1))).collect()
# [('mundo', (<pyspark.resultiterable.ResultIterable at 0x113d82fd0>, <pyspark.resultiterable.ResultIterable at 0x113d82810>)),
#  ('hola',  (<pyspark.resultiterable.ResultIterable at 0x113d824d0>, <pyspark.resultiterable.ResultIterable at 0x113d82f50>))]

dist_data.union(dist_data).map(lambda x: (x,1)).cartesian(udist_data.map(lambda x: (x,1))).collect()
# [(('hola', 1), ('hola', 1)),
#  (('hola', 1), ('mundo', 1)),
#  (('hola', 1), ('hola', 1)),
#  (('hola', 1), ('mundo', 1)),
#  (('mundo', 1), ('hola', 1)),
#  (('mundo', 1), ('mundo', 1)),
#  (('mundo', 1), ('hola', 1)),
#  (('mundo', 1), ('mundo', 1)),
#  (('hola', 1), ('hola', 1)),
#  (('hola', 1), ('mundo', 1)),
#  (('hola', 1), ('hola', 1)),
#  (('hola', 1), ('mundo', 1)),
#  (('mundo', 1), ('hola', 1)),
#  (('mundo', 1), ('mundo', 1)),
#  (('mundo', 1), ('hola', 1)),
#  (('mundo', 1), ('mundo', 1))]

```

 También existen funciones para redistribuir nuestras particiones:

 * coalescence(n) para disminuir a n particiones
 * repartition(n) para redistribuir los datos en n particiones


## Acciones

Algunos ejemplos de acciones.

```python
data = [x for x in range(1,100000)]
dist_data = sc.parallelize(data)
# [1, 2, 3, 4, 5]

dist_data.reduce(lambda x,y: x+y+1)
# 5000049998

dist_data.count()
# 99999

dist_data.take(5)
# [1, 2, 3, 4, 5]

dist_data.first()
# 1

dist_data.takeSample(True,10,0)
# [65893, 74029, 1770, 1244, 67435, 19659, 64573, 3304, 657, 67888]

dist_data.takeOrdered(10)
# [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

def isodd(x):
    if x%2==0:
        return ('odd',x)
    else:
        return ('even',x)

def doprint(x):
    print(x)

dist_data.foreach(doprint)
# Imprimer por consola todos los números

dist_data.map(lambda x: isodd(x)).countByKey()
# defaultdict(int, {'even': 50000, 'odd': 49999})
```

## Variables compartidas

Spark proporciona dos tipos limitados de variables compartidas para dos tipos de uso; Broadcast y Accumulators.

Las variables tipo Broadcast nos permiten mantener en una caché de sólo lectura variables en cada máquina en vez de enviar una copia del mismo con cada tareas . Se pueden utilizar , por ejemplo, para dar a cada nodo de nuestro cluster una copia de un gran conjunto de datos de entrada de una manera eficiente.

```python
broadcast_data = sc.broadcast(['hola','mundo'])
broadcast_data.value
# ['hola', 'mundo']
```

Las variables tipo acumulador nos permiten implementar contadores o hacer agregaciones. Spark soporta de forma nativa los acumuladores de los tipos numéricos aunque es posible añadir soporte para nuevos tipos.

```python
accum = sc.accumulator(0)
sc.parallelize([x for x in range(1,100000)]).foreach(lambda x: accum.add(x))
accum
# Accumulator<id=0, value=4999950000>
```
