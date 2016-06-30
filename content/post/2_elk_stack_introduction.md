+++
date = "2016-06-27T19:08:11+02:00"
draft = true
title = "Introducción a ELK - ElasticSearch / Logstash / Kibana"
description = "Descripción de los componentes del Stack ELK y sus principales usos"
slug = "Introduccion-stack-elk"
tags = ['elasticsearch', 'logstash', 'kibana', 'ELK']
+++

Elasticsearch es una potente herramienta open source que nos permite indexar una gran volumen de datos y posteriormente hacer consultas sobre ellos. Está basado en [Lucene](http://lucene.apache.org/) y es muy escalable y rápido, de echo es usado por  compañías con grandes volúmenes de datos como Wikipedia, Linkedin, etc.

![ELK](/images/elk.jpg)

Actualmente junto con Logstash y Kibana forma el stack tecnológico *ELK* que puede ser utilizado para la ingesta, proceso y visualización de grandes volúmenes de datos, os dejo aquí unas cuestiones interesantes sobre su convercencia dentro del mundo big data y su competencia / complemento con Hadoop:

*  [Why do people use Hadoop or Spark when there is ElasticSearch?](https://www.quora.com/Why-do-people-use-Hadoop-or-Spark-when-there-is-ElasticSearch)
* [What are alternatives to building a data warehouse?](https://www.quora.com/What-are-alternatives-to-building-a-data-warehouse)
* [What are typical use cases where you use the ELK stack instead of a Hadoop stack?](https://www.quora.com/What-are-typical-use-cases-where-you-use-the-ELK-stack-instead-of-a-Hadoop-stack)

## Sobre ELK

Básicamente los componentes de este stack lo forman:

* La herramienta de ingesta de datos [logstash](https://www.elastic.co/products/logstash) que en principio era utilizada únicamente para la ingesta de logs, pero que debido a su buena lista de [plugins](https://www.elastic.co/guide/en/logstash/current/input-plugins.html) podemos capturar muchos tipos de datos. Además es una herramienta sencilla de aprender y utilizar.
* La herramienta [Elasticsearch](https://www.elastic.co/products/elasticsearch) basada en Lucene que nos permite indexar, almacenar y buscar sobre los datos ingeridos a través de logstash.
* La herramienta [Kibana](https://www.elastic.co/products/kibana), que nos permite agregar y visualizar cualquier conjunto de datos.

Esta pila es open source, pero podemos encontrar soporte comercial de [Elastic](https://www.elastic.co) que complementa además el stack con alguna aplicaciones interesantes como [Shield](https://www.elastic.co/products/shield) para el control de acceso y seguridad.

## Logstash

Aunque su orientación inicial es la ingesta de logs, este componente permite capturar infinidad de datos de fuentes heterogeneas y realizar filtros complejos sobre estos datos a través de sus [plugins](https://www.elastic.co/guide/en/logstash/current/input-plugins.html).

Para configurar un flujo de ingesta úncamente debemos configurar en un fichero los plugins que utilizemos para:

* Entrada de datos: Logs, Archivos, Twitter, BBDD, Heroku, github ...
* Filtrado: Anonimato, Duplicación, Agregar, Eliminar, Métricas, ...
* Salida: Elasticsearch, CSV, Email, Jira, ...

Permite además capturar diversos tipos de datos en un mismo flujo o utilizar flujos distintos por lo podemos organizar la ingesta de datos según nuestras necesidades.

Se trata de una herramienta autónoma realmente versatil y sencilla de utilizar. Está además bien documentada y en [stackoverflow] (http://stackoverflow.com/) tienes el triple de preguntas sobre logstash que sobre [flume](https://flume.apache.org/).


## Elasticsearch

Elasticsearch tiene una arquitectura distribuida lo que le permite escalar horizontalmente y añadir distintos tipos de nodos en nuestra arquitectura.

Algunos términos importantes a manejar:

* **Indices**: Toda la información almacenada en elasticsearch está almacenada en forma de indices, por lo que la captura de datos se llama directamente indexar.
* **Fragmentos**: Todos los índices son almacenados en fragmentos que corresponden a una base de datos de Lucene. En cada nodo tendremos al menos un fragmento.
* **Réplicas**: Son fragmentos que se repiten en distintos nodos para mantener alta disponibilidad.
* **Tipos de Nodos**:
  * Data Node - Contienen los datos y fragmentos, su función principal es almacenar los datos y realizar las operaciones de busqueda, CRUD y agregacion.
  * Client Node - Actua como enrutador inteligente de solicitudes
  * Master Node - Actua gestionando el cluster de nodos, puede actuar también como Data Node o sólo como Master Node.


## Kibana

Kibana es una herramienta que nos permite crear un entorno analítico completo en tiempo real con un interfaz realmente intuitivo. Permite además tanto la configuración de distintos paneles y analíticas como el descubrimiento y búsqueda de datos en tiempo real.   


## Dimensionado

La cantidad de recursos que necesitaremos vendrá determinada por la cantidad de datos que vamos a procesar y el tipo de búsquedas e información a agregar y visualizar.

Para calcular el número de fragmentos y nodos a utilizar debemos primero calcular la capacidad sobre un nodo y después extrapolar con datos reales y nuestros requisitos, más o menos con un flujo similar al siguiente:

1. Creamos un cluster en un nodo con unas características hardware similares a las de produción.
2. Configuramos un proceso de indexado similar al que finalmente irá en producción en un fragmento sin réplicas.
3. Insertamos datos reales (lo más reales posible) y realizamos búsquedas y agregaciones (lo más reales posibles).

A partir de aqui calculamos los tiempos y extrapolamos con el conjunto de datos completo y los tiempos deseados.

Con respecto al resto de componentes (logstash y Kibana) al ser autónomos pueden ir en el nodo cliente o en nodos separados, dependiendo de las necesidades específicas.


## En breve

En el próximo artículo vamos a instalar un cluster ELK en Amazón Web Services y ejecutar una captura de datos de twitter. Lo tenéis [aquí](/post/Instalacion-cluster-ELK-Elasticsearch-logstash-kibana/)
