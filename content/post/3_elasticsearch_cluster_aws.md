+++
date = "2016-06-28T19:08:11+02:00"
draft = true
title = "Instalación de un cluster ELK - ElasticSearch / Logstash / Kibana"
description = "Tutorial rápido para montar un cluster ELK sobre Amazon Web Services y un pequeño ejemplo de ingesta a través de twitter"
slug = "Instalacion-cluster-ELK-Elasticsearch-logstash-kibana"
tags = ['elasticsearch', 'logstash', 'kibana', 'AWS']
+++

En este turorial vamos a mostrar como montar un cluster [ELK](post/Introduccion-stack-elk/) en Amazon Web Services por menos de 1€ con la siguente arquitectura:

* Nodo Cliente + Logstash + Kibana
* Nodo Master
* Nodo Data

## Sobre ELK

Elasticsearch es una potente herramienta open source que nos permite indexar una gran volumen de datos y posteriormente hacer consultas sobre ellos. Está basado en [Lucene](http://lucene.apache.org/) y es muy escalable y rápido, de echo es usado por  compañías con grandes volúmenes de datos como Wikipedia, Linkedin, etc.

![ELK](/images/elk.jpg)

Actualmente junto con Logstash y Kibana forma el stack tecnológico *ELK* que puede ser utilizado para la ingesta, proceso y visualización de grandes volúmenes de datos.

## Lanzando nuestras instancias sobre AWS / EC2 Spot Instances

Las denominadas instancias "puntuales" (Spot instances) de Amazon nos permiten probar muchas arquitecturas de un modo mucho más barato que con instancias normales. Funcionan pujando sobre una serie de instancias libres a un precio sensiblemente inferior de hasta un 80%, la "pega" es que si hay demanda de ese tipo de instancias con un precio superior a tu puja puedes perder la instancia de forma casi instantánea (2 minutos).

No obstante si realizas una puja con un ahorro del 50% (en vez del 80% máximo) sueles tener instancias para rato.

Para seleccionarlas simplemente en la consola EC2 > Spot Instances puedes reservar 3 instancias que son las que vamos a necesitar, adjunto un pantallazo de mi reserva:

![Reserva AWS - EC2](/images/3_aws1.png)

Es importante que crees si no tienes un certificado para hacer ssh a nuestras máquinas y un grupo de seguridad que permita en este caso la entrada y salida por cualquier puerto para esta prueba.

La instalación básica es realmente sencilla y sigue los siguientes pasos:

* Modificar el fichero */etc/hosts* para nombrar las direcciones de nuestras máquinas
* Bajar e instalar en todas las máquinas el paquete Elasticsearch
* Modificar el fichero */etc/elasticsearch/elasticsearch.yml* de cada nodo para configurar su rol y la red


## Instalación del cluster Elasticsearch

Comenzando abriendo 3 consolas, una por cada instancia, y tomamos nota de las direcciones IP's internas para añadir en el fichero */etc/hosts* de **todas** las instancias los nodos con las IP's, pongo un ejemplo con las direcciones que tengo asignadas.

```zsh
172.31.8.109    es-client-kibana
172.31.10.33    es-master
172.31.10.222   es-data
```

Ahora necesitamos instalar java 8 si es que no lo tenemos ya instalado.

```zsh
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
java -version
```

Para instalar Elasticsearch descarga el fichero e instalaló. Pongo aqui unas instrucciones para elasticsearch_2.3.3 y ubuntu, si tienes otra distribucion puedes consultar directamente en su página [aquí](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-repositories.html)

```zsh
wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/deb/elasticsearch/2.3.3/elasticsearch-2.3.3.deb
sudo dpkg -i elasticsearch-2.2.0.deb
sudo service elasticsearch start
```

Si queréis que el servicio se inicie automáticamente en el arranque podemos utilizar el siguiente comando.

```zsh
sudo update-rc.d elasticsearch defaults 95 10
```

Ahora para verificar la instalación vamos a enviar una solicitud http al puerto 9200 (puerto por defecto de elasticsearch) donde recibiréis respuesta con vuestros datos.

```zsh
curl http://localhost:9200

{
  "name" : "es-data",
  "cluster_name" : "es-demo",
  "version" : {
    "number" : "2.3.3",
    "build_hash" : "218bdf10790eef486ff2c41a3df5cfa32dadcfde",
    "build_timestamp" : "2016-05-17T15:40:04Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.0"
  },
  "tagline" : "You Know, for Search"
}
```

Como comentamos, toda la configuración de elasticsearch está en */etc/elasticsearch/elasticsearch.yml*. En cada uno de los ficheros configuraremos:

* El nombre del cluster
* El rol del nodo con tres variables booleanas (node.client, node.data y node.master)
* Nuestra dirección IP
* Nodos de nuestro cluster en el nodo cliente

Os dejo aquí la configuración que estamos siguiendo

### Nodo Cliente

Este nodo actuará como enrutador inteligente de solicitudes

```zsh

cluster.name: es-demo

node.name: es-client-kibana
node.client: true
node.data: false

network.host: 172.31.8.109

discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["es-client-kibana", "es-master",  "es-data"]

```

### Nodo Master

Este nodo actuará gestionando el cluster de nodos, puede actuar también como Data Node o sólo como Master Node pero en nuestro caso sólo lo hará como Master Node.

```zsh

cluster.name: es-demo

node.name: es-master
node.master: true
node.data: false

network.host: 172.31.10.33

discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["es-client-kibana", "es-master",  "es-data"]

```

### Nodo Data

Este nodo contiene los datos y su función principal es almacenar los datos y realizar las operaciones de busqueda, CRUD y agregacion.

```zsh

cluster.name: es-demo

node.name: es-data
node.client: false
node.data: true

network.host: 172.31.10.222

discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["es-client-kibana", "es-master",  "es-data"]
```

Ahora reiniciamos los tres servicios y ya tendremos nuestro cluster montado.

```zsh
sudo service elasticsearch start
```

Podemos ver el estado de nuestro cluster con otra solicitud HTTP

```zsh
curl http://es-client-01:9200/_cluster/stats
```

## Estatus de nuestro cluster

Para revisar el estado de nuestro cluster con facilidad podemos instalar una herramienta en nuestro nodo cliente. Para esto debemos ir a */usr/share/elasticsearch/bin* y ejecutar el siguiente comando.

```zsh
./plugin install mobz/elasticsearch-head
```

Reinicia el servicio para que funcione el plugin accede a [http://:9200/_plugin/head/](http://:9200/_plugin/head/) en tu navegador para ver el detalle del cluster.


## Instalación de Logstash

Vamos a instalar esta herramienta en el nodo cliente. En su consola debemos bajar el paquete e instalarlo. Pondré aquí las instrucciones para Ubuntu, aunque si tienes otra distribución puedes revisar la [documentación](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html)

```zsh
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://packages.elastic.co/logstash/2.3/debian stable main" | sudo tee -a /etc/apt/sources.list
sudo apt-get update && sudo apt-get install logstash
```

Ahora para probar que todo se ha instalado correctamente puedes dirigirte al directorio de Logstash (en ubuntu */opt/logstash*) y ejecutar la siguiente prueba, que sacará por consola todo lo que escribas en la misma.

```zsh
bin/logstash -e 'input { stdin { } } output { stdout {} }'
```

## Instalación de Kibana

De igual modo instalaremos Kibana en nuestro nodo cliente. Pondré aquí las instrucciones para Ubuntu, aunque si tienes otra distribución puedes revisar la [documentación](https://www.elastic.co/guide/en/kibana/current/setup.html)

```zsh
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb http://packages.elastic.co/kibana/4.5/debian stable main" | sudo tee -a /etc/apt/sources.list
sudo apt-get update && sudo apt-get install kibana
```

Ahora para entrar en Kibana simplemente en el navegador dirígete al puerto 5601 de la IP pública (revisa AWS) del nodo cliente.


## Qué hago ahora?

Desde aquí te proponemos para probar la instalación un ejemplo típico:

* Ingesta de datos desde twitter a través de Logstash
* Indexado en ElasticSearch
* Visualización en Kibana

Lo primero es crear una aplicación en Twitter, queda fuera de este tutorial pero es realmente sencillo. Dirígete a [twitter](https://dev.twitter.com/) y crea una aplicación, puedes encontrar ayuda [aquí](http://www.vozidea.com/crear-una-aplicacion-en-twitter-para-usar-la-api)

Ahora con los datos de twitter creamos una configuración de *Logstash* que capture los datos de twitter (tweets con la palabra spain) y lo almacenamos en el fichero "spain.conf". Sustituye aquí los datos de tu aplicación en consumer_key, consumer_secret, access_token, access_token_secret.

```zsh

input {
  twitter {
      consumer_key => "consumer_key"
      consumer_secret => "consumer_secret"
      oauth_token => "access_token"
      oauth_token_secret => "access_token_secret"
      keywords => [ "spain" ]
      full_tweet => true
  }
}
filter {
}
output {
  stdout { codec => dots }
  elasticsearch {
    protocol => "http"
    host => "localhost"
    index => "twitter"
    document_type => "tweet"
    template => "twitter_template.json"
    template_name => "twitter"
  }
}
```

Es importante aquí reflejar el nombre del índice que estamos creando, en este caso `index => "twitter"` por que después lo buscaremos con Kibana.

Para indexar los datos necesitaremos conocer la estructura de los tweets, como véis hago referencia a esta estructura en *template* con un fichero. Este fichero "twitter_template.json" es necesario crearlo y su contenido es el siguiente.

```zsh
{
  "template": "twitter",
  "order":    1,
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "tweet": {
      "_all": {
        "enabled": false
      },
      "dynamic_templates" : [ {
         "message_field" : {
           "match" : "message",
           "match_mapping_type" : "string",
           "mapping" : {
             "type" : "string", "index" : "analyzed", "omit_norms" : true
           }
         }
       }, {
         "string_fields" : {
           "match" : "*",
           "match_mapping_type" : "string",
           "mapping" : {
             "type" : "string", "index" : "analyzed", "omit_norms" : true,
               "fields" : {
                 "raw" : {"type": "string", "index" : "not_analyzed", "ignore_above" : 256}
               }
           }
         }
       } ],
      "properties": {
        "text": {
          "type": "string"
        },
          "coordinates": {
          "properties": {
             "coordinates": {
                "type": "geo_point"
             },
             "type": {
                "type": "string"
             }
          }
       }
      }
    }
  }
}
```

Una vez salvados ambos ficheros podemos ejecutar *Logstash* y podremos ver que empieza a capturar tweets

```zsh
nohup bin/logstash -f spain.conf &

# Settings: Default pipeline workers: 4
# Pipeline main started
# ...........................................................................
```

Ahora en Kibana debemos configurar un nuevo índice, sustituyendo logstash-* por nuestro indice *twitter* previamente configurado en logstash

![Kibana Index](/images/3_kindex.png)

*Et voila* ya tenemos datos en tiempo real. Cualquier duda podéis contactarme directamente en el correo. 
