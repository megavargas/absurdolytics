+++
date = "2016-06-30T19:08:11+02:00"
draft = true
title = "Instalar Zeppelin en Mac"
description = "Montar un notebook Zeppelin en Mac"
slug = "zeppelin-mac"
tags = ['zeppelin', 'mac']
+++


Aunque en muchas de las pruebas y demos que hacemos en [{contextual}](http://contextual.team) las hacemos sobre *spot* instancias de Amazon, para pruebas sencillas y cacharreo suelo utilizar directamente mi portatil, tanto con docker como diréctamente sin virtualización.

En esta ocasión os dejo como instalar fácilmente Zeppelin directamente sobre Mac. Instalar Zeppelin es bastante màs complejo que Jupyter, pero con estas instrucciones resulta mucho más sencillo.

Lo primero, si no lo tenéis instalado ya, es instalar Maven. Esto es sencillo y directo a través de brew.

```zsh
brew install maven
```

Ahora es necesario obtener [Zeppelin](https://github.com/apache/incubator-zeppelin/archive/master.zip), descomprimirlo, abrir un terminal, instalarlo y lanzarlo.

```zsh
cd ~/Downloads/zeppelin-master
mvn clean install -Pspark-1.6 -DskipTests
bin/zeppelin-daemon.sh start
```

Para acceder a Zeppelin simplemente dirígete a http://localhost:8080 y ya estaría

![Zeppelin](/images/4_zeppelin.png)
