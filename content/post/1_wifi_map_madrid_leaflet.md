+++
date = "2016-06-19T19:08:11+02:00"
draft = true
title = "Leaflet.js / OpenStreetMap y Datos abiertos de Madrid <Javascript>"
description = "Una visualización utilizando leaflet.js / OpenStreetMap y datos abiertos de Madrid"
slug = "Instalaciones-municipales-con-zonas-wifi-gratuitas-leaflet-OpenStreetMap-Madrid"
tags = ['leaflet', 'mapas', 'OpenStreetMap']
+++

En este post vamos a realizar una integración sencilla que nos va a permitir visualizar los puntos wifi municipales abiertos en Madrid.

Para el mapa utilizaremos [leaflet.js](http://leafletjs.com/), una librería javascript Open Source que nos permite crear mapas interactivos en entornos móviles. Los datos los hemos obtenido del [portal de datos abiertos de Madrid](http://datos.madrid.es/) y los hemos transformado en JSON utilizando una herramienta online para simplificar este ejemplo.

Vamos al código.

## Preparando la página

En el *head* de la página incluye la hoja de estilos a través de un CDN
```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/leaflet.css">
```

Incluye al final la librería javascript
```html
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/leaflet.js"></script>
```

Además en el *body* debemos incluir un div donde se insertará el mapa. Aquí para facilitar el ejemplo se ha incluído un estilo *inline* para las dimensiones del mapa.
```html
<div id="mapid" style="height: 280px"></div>
```

## Configurando el mapa

Vamos a crear un mapa de Madrid partiendo de la Puerta del Sol (cuyas coordenadas son LAT 40.415363, LON -3.707398 ) y un zoom inicial que nos permita verlo entero. Aquí es donde enlazamos con el *div* previamente creado a través del id *mapid*

```javascript
var mymap = L.map('mapid').setView([40.415363, -3.707398], 10);
```

A continuación se añade un mapa base como *tile layer* utilizando las imágenes de [OpenStreetMap](https://www.openstreetmap.org/). Para generar el mapa a través de OpenStreetMap se debe incluir la URL, el texto con la atribución y el zoom máximo.

```javascript
L.tileLayer('http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: 'Map data &copy; <a href="http://openstreetmap.org">OpenStreetMap</a> contributors, <a href="http://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="http://cloudmade.com">CloudMade</a>',
    maxZoom: 18
}).addTo(mymap);
```

Ahora insertamos los datos obtenidos del [portal de datos abiertos de Madrid](http://datos.madrid.es/) que previamente hemos transformado a JSON y cargado en *wdata*

```javascript
for (var i=0; i<wdata.length; i++) {
    var circle = L.circle([wdata[i].LATITUD,wdata[i].LONGITUD], 10, {
    color: 'red',
    fillColor: '#f03',
    fillOpacity: 0.5
    }).addTo(mymap);      

    circle.bindPopup(wdata[i].NOMBRE);  
}
```

El resultado podéis verlo a continuación

<div id="mapid" style="height: 280px"></div>


<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/leaflet.js"></script>
<script src="/rawdata/wdata.js"></script>

<script type="text/javascript">

    function loadjscssfile(filename, filetype) {
            if (filetype == "js") {
                var fileref = document.createElement('script')
                fileref.setAttribute("type", "text/javascript")
                fileref.setAttribute("src", filename)
                alert('inserted')
            }
            else if (filetype == "css") { //if filename is an external CSS file
                var fileref = document.createElement("link")
                fileref.setAttribute("rel", "stylesheet")
                fileref.setAttribute("type", "text/css")
                fileref.setAttribute("href", filename)
            }
            if (typeof fileref != "undefined")
                document.getElementsByTagName("head")[0].appendChild(fileref)
        }

    loadjscssfile("https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/leaflet.css","css");

    var mymap = L.map('mapid').setView([40.415363, -3.707398], 10);

    L.tileLayer('http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: 'Map data &copy; <a href="http://openstreetmap.org">OpenStreetMap</a> contributors, <a href="http://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="http://cloudmade.com">CloudMade</a>',
        maxZoom: 18
    }).addTo(mymap);

    for (var i=0; i<wdata.length; i++) {
        var circle = L.circle([wdata[i].LATITUD,wdata[i].LONGITUD], 10, {
        color: 'red',
        fillColor: '#f03',
        fillOpacity: 0.5
        }).addTo(mymap);      

        circle.bindPopup(wdata[i].NOMBRE);  
    }

</script>

Hemos aprovechado para añadir un popup con el nombre del centro que provee de wifi.

Podéis ver muchos ejemplos y [tutoriales](http://leafletjs.com/examples.html) en la página oficial de [leaflet.js](http://leafletjs.com/)
