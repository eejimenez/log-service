# Registro de Logs
 
El servicio de logs estará compuesto por estas herramientas:
 
- **Fluentd**
     
    Se dedica a recoger toda la información o logs que generan las diferentes aplicaciones en un sistema de software y enviarlo o almacenarlo en diferentes fuentes.
 
- **Sentry**
 
    Es un sistema de seguimiento de errores de código abierto compatible con diferentes frameworks y lenguajes de programación.
 
 
## Arquitectura
 
![Arquitectura!](/img/log-service.png "Arquitectura")
 
Fluentd será el encargado de recopilar los logs de diferentes orígenes utilizando diferentes protocolos de comunicación y luego los enviará a Sentry u otros destinos que se definan.
 
## Fluentd
 
Fluentd funciona por medio de uno o varios archivos de configuración en donde se definen las entradas de información, filtros y destinos.
 
Fluentd ofrece nativamente las siguientes entradas:
 
- tail
 
    Lee eventos en un archivo de texto.
 
- forward
 
    Escucha un socket TCP para recibir el flujo de eventos.
 
- udp
 
    Lee informacion a traves del protocolo UDP
 
 
- tcp
 
    Lee informacion a traves del protocolo TCP
 
- unix
 
    Recupera registro del socket de dominio Unix
 
- http
 
    Crea un api REST para recopilar datos
 
Entre otras.
 
Además dispone de varios plugins para integrarse con otras plataformas o servicios.
 
https://www.fluentd.org/plugins/all
 
 
### Configuración
 
El archivo de configuración de fluentd trabaja con una sintaxis de etiquetas en donde se definen las entradas, filtros y destinos.
 
Por Ejemplo:
 
```
<source>
...
</source>
 
<match name.name>
...
</match>
```
 
Las directivas disponibles son las siguientes:
 
- **source**
 
    Indica de dónde viene la información
 
- **match**
 
    indica qué hacer con la información
 
- **filter**:
 
    Indica un flujo de procesamiento
 
- **system**
 
- **label**
 
    Agrupa directivas de filter y match
 
 
- **@include**
 
    Importa configuración de otro archivo
 
### Plugins
 
Como se mencionó antes existen varios plugins que permiten la integración de fluentd con otras herramientas para obtener entradas y salidas.
 
Para agregar un plugin se debe agregar la instalación del plugin en el archivo Dockerfile de fluentd.
 
Por ejemplo:
 
`RUN gem install fluent-plugin-sentry`
 
Esto nos permite utilizar una directiva **match** de tipo **sentry**.
 
```
<match notify.**>
  @type sentry
 
  # Set endpoint API URL
  endpoint_url       http://publicKey:privateKey@host:port/projectId
 
  # Set default events value of 'server_name'
  # To set short hostname, set like below.
  hostname_command   hostname -s
 
  # rewrite shown tag name for Sentry dashboard
  remove_tag_prefix  notify.
</match>
```
 
 
### Instalación
 
Construir y ejecutar la imagen de docker ubicada en la carpeta **fluentd**
 
```
docker build -t cartful/fluentd .
docker run -p 24224:24224 cartful/fluentd
```
 
 
## Sentry
 
### Instalación
 
Ofrece un repositorio donde se encuentran todos los archivos necesarios para la instalación.
 
**Requerimientos**
 
- Docker 19.03.6+
- Compose 1.28.0+
- 4 CPU Cores
- 8 GB RAM
- 20 GB de almacenamiento
 
**Pasos para instalación**
 
1. Clonar el repositorio de sentry self-hosted
 
    `git clone https://github.com/getsentry/self-hosted.git`
 
2. Ejecutar el archivo *install.sh*
 
    `./install.sh`
 
3. Ejecutar sentry con docker-compose
 
    `docker-compose up -d`
 
## Enviar logs a fluentd
 
### Node Js
 
Existe la librería [@fluent-org/logger](https://github.com/fluent/fluent-logger-forward-node) que implementa el protocolo forward de fluentd.
 
`npm install @fluent-org/logger`
 
Agregar la siguiente directiva en el archivo fluent.conf
 
```aconf
<source>
  @type forward
  port 24224
</source>
```
 
Agrega
 
```js
const FluentClient = require("@fluent-org/logger").FluentClient;
const logger = new FluentClient("tag_prefix", {
  socket: {
    host: "localhost",
    port: 24224,
    timeout: 3000, // 3 seconds
  }
});
```
 
### Docker
 
Docker cuenta con logging driver fluentd, el cual enviará logs del contenedor a un colector fluentd.
 
`docker run --log-driver=fluentd --log-opt fluentd-address=fluentdhost:24224`
 
### Http
 
Al crear una directiva source de tipo http se habilitará un API que recibirá los logs.
 
```
<source>
  @type http
  port 9880
  bind 0.0.0.0
  body_size_limit 32m
  keepalive_timeout 10s
</source>
```
 
Ejemplo de uso básico
 
```
# Post a record with the tag "app.log"
$ curl -X POST -d 'json={"foo":"bar"}' http://localhost:9880/app.log
```

