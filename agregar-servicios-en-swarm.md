# AGREGAR SERVICIOS A UN SWARM

[Docker swarm-services](https://docs.docker.com/engine/swarm/services/)

## DESCRIPCIÓN DEL COMANDO DOCKER SERVICE CREATE

```bash
docker service create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

[Descripción completa](https://docs.docker.com/engine/reference/commandline/service_create/)

**NOTA**: La _IMAGE_ por lo general se descarga desde el repositorio de [DockerHub](https://hub.docker.com/), si esta no se encuentra en las imagenes almacenadas en el nodo en donde se ejecutara el servicio y su respectivo contenedor.

## CREAR UN SERVICIO

A continuación se muestra la creación de un servicio con un única replica y sin ninguna configuración adicional. Este comando inicia el servicio de MongoDB con un nombre aleatorio y sin publicar puertos.

Para esta guía se trabaja sobre el Swarm inicializado en los nodos creados en VirtualBox.

* Ingresar al manager:

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter  $ ]**[command docker-machine ssh manager1]
```

* Crear el servicio de Mongo:
 
```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter  $ ]**[command docker service create mongo]
```

Este servicio se levanta en un nodo del Swarm. Para verificar que el servicio fue creado e inicializado de forma correcta, se usa el siguiente comando:

```
$ docker node ls

ID                  NAME                MODE                REPLICAS            IMAGE                                                                                             PORTS
a3iixnklxuem        quizzical_lamarr    replicated          1/1                 docker.io/library/nginx@sha256:41ad9967ea448d7c2b203c699b429abe1ed5af331cd92533900c6d77490e0268
```

Al crear un servicio puede que no se ejecute de forma correcta. Puede quedarse en estado pendiente si por ejemplo la imagen solicitada no es valida, si el nodo no reconoce los requisitos de configuración del servicio, etc.

**Para agragarle un nombre al servicio, se usa la bandera --flag**:

```
$ docker service create --name my_web nginx
```

**Para especificar un comando que el contenedor debe ejecutar, este se añado despues del nombre de la imagen**:

```
$ docker service create --name helloworld alpine ping docker.com
```

**alpine** es el nombre de la imagen y **ping docker.com** es el comando a ejecutar en esta imagen.

**Para especificar una imagen particular a usar como servicio se puede agregar un tag despues del nombre de la misma**:

```
$ docker service create --name helloworld alpine:3.6 ping docker.com
```

## LISTAR SERVICIOS

```
$ docker service ls
```

## ACTUALIZAR UN SERVICIO

Es posible actualizar cada parametro de un servicio existente usando el comando **docker service update**. Cuando se actualiza un servicio, docker detiene el contenedor y lo reinicia el servicio con la nueva configuración.

Anteriormente se creo un servicio del servidor Nginx, pero no tiene expuesto el puerto 80 y por tanto no se es util en el mundo real. Al crear el servicio se puede especificar el puerto usando la bandera **-p** o **--publish**.

```
$ docker service create --name my_web --publish 80 nginx
```

Cuando se desea actualizar el servicio se debe usar la bandera **--publish-add**:

```
$ docker service update --publish-add 80 my_web
```

Para remover un puerto previamente publicado se utiliza la bandera **--publish-rm**:

```
$ docker service update --publish-rm 80 my_web
```

## REMOVER UN SERVICIO

Para remover un servicio se usa el comando **docker service remove**. El servicio puede ser removido usando su ID o su nombre, estos se muestran en la salida del comando **docker service ls**.

```
$ docker service remove my_web
```



