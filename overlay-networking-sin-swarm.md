# Red Overlay sin Swarm

Una de las cosas que se debe tener en cuenta al trabajar con multiples host en Docker es que cada uno de ellos no pueden verse entre sí. Por lo cual, no se le es permitido a los host compartir ningún recurso, ni siquiera una red. Pero existe una forma de hacerlo y es crear un servicio especial que tenga como única tarea mantener una lista de host involucrados, la configuración de la red y que le diga a los _Do\_ocker Engine_ \_como deben usarla.

Para la creación de el servicio mencionado anteriormente, se puede trabajar con varios _discovery services tales como Consul, ZooKeeper _ó_ Etcd_. Para esta guía se trabajará con el servicio [_**Consul**_](https://www.consul.io/).

## VirtualBox

La primera parte de esta guía se hara generando las máquinas que serviran como host sobre _VirtualBox_ usando la herramienta _docker-machine._

### Prerrequisitos

* VirtualBox: En donde se correran los host virtuales
* docker-machine: Usando para crear los host. Si se esta ejecutando Docker sobre Mac o Linux, probablemente este ya se encuentre instalado. De otro modo, las instrucciones de instalación se encuentran en el siguiente [link](https://docs.docker.com/machine/install-machine/).

### Instalación del _Discovery Service_

Lo primero es crear un _host_ con Consul ejecutandose en un contenedor. Lo primero es crear de _host_ de la siguiente forma:

```
docker-machine create -d virtualbox keystore
```

Basicamente se le dice a _docker-machine_ que debe crear un _host_ llamado **keystore** usado el driver de VirtualBox. El _host_ se creará con _Docker Engine_ totalmente configurado, lo cual permitirá descargar y ejecutar la imagen de _Consul_.

Docker permite dos formas de conectarse al _host_ creado. La primera de ellas es usar \_**docker-machine ssh &lt;nombre\_maquina&gt; **\_con lo cual se accede de forma directa al host. La segunda forma, permite conectar el  Docker Engine local con el del host remoto, en otras palabras, los comandos ejecutados localmente se ejecutaran sobre el host.

Para realizar la conección del Docker Engine local al del host remoto:

```
eval $(docker-machine env keystore)
```

De aquí en adelante cualquier comando que se realice con _**docker**_ realmente se ejecutaŕa sobre la máquina keystore corriendo en VirtualBox.

Para crear el contenedor de Consul en la máquina keystore se ejecuta:

```
docker run --name Consul -d -p 8500:8500 progrium/consul -server -bootstrap
```

Cabe hacer notar que el puerto 8500 del contenedor es mapeado al puerto 8500 del host. Para obtener la dirección ip del host y verificar el funcionamiento de Consul se ejcuta:

```
docker-machine ip keystore
    192.168.99.104
```

Desde un navegador puede entrarse a la dirección ip resultante de la ejecuación del comando anterior junto con el puerto 8500 \([http://192.168.99.104:8500](http://192.168.99.104:8500)\).

### Configuración de los host con Docker Engine

Para este tutorial se crearán dos host con Docker Engine que conozcan el _**discovery service **_** \(Consul\) **que se ha creado. Docker Engie tiene dos propiedades para el modo cluster:

1. **cluster-store: **Apunta al _discovery service_
2. **cluster-advertise: **Especifica una puerta de conexión del actual Docker Engine que permita a otros Engine comunicarse. Generalmente _docker-machi_ne_ lo crea en los host en **eth1:2376**._

Para la creación del host **node-0** se ejecuta:

```
docker-machine create -d virtualbox \
 --engine-opt="cluster-store=consul://$(docker-machine ip keystore):8500" \
 --engine-opt="cluster-advertise=eth1:2376" \
 node-0
```

Para la creación del host **node-1** se ejecuta:

```
docker-machine create -d virtualbox \
 --engine-opt="cluster-store=consul://$(docker-machine ip keystore):8500" \
 --engine-opt="cluster-advertise=eth1:2376" \
 node-1
```

Despues de ejecutados estos comandos, se crean los dos host conectados al servicio Consul. Los Docker Engine de cada uno de los host se comunican a traves del puerto 2376.

### Creación de la red Overlay

Para la creación de la red Overlay se conectará a uno de los nodos mediante el metodo SSH.

```
docker-machine ssh node-0
```

Una vez conectado a esta máquina se creará la red _Overlay_.

```
docker@node-0:~$ docker network create -d overlay multi-host-net
```

Para comprobar que esta red es visible desde ambos _hosts, _ se desconetara del _node-0, _ se conectará al _node-1 y se ejecutará el siguiente comando:_

```
docker network ls
    #NETWORK ID          NAME                DRIVER              SCOPE
    #42131dca03d6        bridge              bridge              local
    #46793b96eaef        host                host                local
    #e1d516c32c3c        multi-host-net      overlay             global
    #c68e3ec2231b        none                null                local
```

Como se observa, la red overlay llamada **multi-host-net** es visible desde el host **node-1** así esta halla sido creada desde el host **node-0**.

### Test de conexión sobre la red Overlay

Desde el host **node-1** se creará un contenedor con **NGIX** enlazado al ared overlay que se creo anteriormente.

```
docker run -d -p80:80 --net=multi-host-net --name=ngixserver nginx
```

El contenedor en este caso se denomino **ngixserver** y este nombre será usado como nobre de dominio. Para verficar que el servicio se esta ejecutando correctamente se ejecuta:

```
curl localhost
```

Este comando debe devolver la página de bienvenida del servidor NGIX.

Ahora se desconectará el host **node-1** y sobre el host **node-0** se crearán un contenedor con ubuntu para acceder al servicio de ngix ejecutandose en un contenedor del host **node-1**.

Conectarse al host **node-0**:

```
docker-machine ssh node-0
```

Crear el contenedor con ubuntu e ingresar a una terminal interactiva:

```
docker@node-0:~$ docker run -ti --net=multi-host-net ubuntu bash
```

Actualizar los repositorios e instalar curl:

```
apt-get update && apt-get install -y curl
```

Realizar una petición al servicio de NGIX

```
curl http://ngixserver

    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>
    
    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>
    
    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
```

Como se puede observar, un contenedor ejecutandose en el host **node-0** tiene la habilidad de alcanzar el contenedor ngixserver en el host **node-1** usando solo el nombre del contenedor.







