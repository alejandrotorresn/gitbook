# AGREGAR SERVICIOS A UN SWARM

## CREAR UN SERVICIO

Creación de un servicio con un única replica y sin ninguna configuración adicional. Este comando inicia el servicio de Nginx con un nombre aleatorio y sin publicar puertos.

```
$ docker service create nginx
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

## ACTUALIZAR UN SERVICIO





