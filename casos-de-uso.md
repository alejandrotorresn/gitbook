# SWARM MODE - IMPLEMENTACIÓN EN ANÁLITICA DE DATOS

Para los casos de uso se implementará un solo nodo **manager **y dos nodos **workers** mediante **docker-machine** y en _VirtualBox_.

Se implementaran los servicios de **MongoDB**, Servidores REST basados en **Eve** junto con una configuración personalizada de **Apache** que permite recibir archivos .gzip en el servicio REST y servicios **Analítica de Datos** \(Jupyter Notebook y Jupyter Lab\).

## REQUISITOS PREVIOS

1. [Instalación de Docker Engine](/instalacion.md)
2. [Instalación de Docker Machine](/instalacion-de-docker-machine.md)
3. Creación de las máquinas virtuales en _VirtualBox_ 

**NOTA:** Los dos primeros requisitos pueden satisfacerse siguiendo los enlaces. No es necesario la instalación de _Docker Engine_ si se salta directamente a los nodos mediante **docker-machine ssh < node_name >** y no se usa **docker-machine env < node_name >** para conectar el cliente Docker al Docker Engine corriendo en la máquina virtual.

## CREACIÓN DE LAS MÁQUINAS VIRTUALES

* Creación del nodo Manager \(Es recomendable asignarle un mínimo de 4GB de memoria al nodo manager\):

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine create -d virtualbox --virtualbox-memory 4096 manager1]
Creating CA: /home/user/.docker/machine/certs/ca.pem
Creating client certificate: /home/user/.docker/machine/certs/cert.pem
Running pre-create checks...
(manager1) Image cache directory does not exist, creating it at /home/user/.docker/machine/cache...
**[warning (manager1) No default Boot2Docker ISO found locally, downloading the latest release...]
(manager1) Latest release for github.com/boot2docker/boot2docker is v17.10.0-ce
(manager1) Downloading /home/user/.docker/machine/cache/boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v17.10.0-ce/boot2docker.iso...
(manager1) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
Creating machine...
(manager1) Copying /home/user/.docker/machine/cache/boot2docker.iso to /home/user/.docker/machine/machines/manager1/boot2docker.iso...
(manager1) Creating VirtualBox VM...
(manager1) Creating SSH key...
(manager1) Starting the VM...
(manager1) Check network to re-create if needed...
(manager1) Found a new host-only adapter: "vboxnet0"
(manager1) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env manager1
```
 **SALIDA DEL COMANDO DOCKER-MACHINE CREATE**

 **NOTA:** En la salida de la terminal puede observarse que no se encontro la imagen localmente y se procede a descargarla del repositorio de DockerHub-


* Creación de los nodos Worker

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine create -d virtualbox worker1]
Running pre-create checks...
Creating machine...
...
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env worker1
```

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine create -d virtualbox worker2]
Running pre-create checks...
Creating machine...
...
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env worker2
```



## CONFIGURACIÓN DE SWARM 

## CONFIGURACIÓN DEL ENTORNO DE ANÁLITICA DE DATOS

## CASOS DE USO




### CASO DE USO BÁSICO - DESPLIEGUE DE SERVICIOS DE ANÁLITICA


### CASOS DE USO - AVANZADO







### CREACIÓN DE LAS MÁQUINAS VIRTUALES



### CREACIÓN DEL SWARM

Antes de crear el Swarm se recomienda establecer la dirección IP del nodo manager como dirección estática, para ello se debe:

* Obtener la dirección dada por el servidor DHCP a la máquina **manager** creada por docker-machien.

```
$ docker-machine ip manager
```

* Ingresar al nodo **manager**.

```
$ docker-machine ssh manager
```

* Crear el archivo **bootsync.sh**

```
$ sudo vi /var/lib/boot2docker/bootsync.sh
```

* Ingresar los siguientes comandos en el archivo bootsync.sh

```
#!/bin/sh
/etc/init.d/services/dhcp stop
ifconfig eth1 192.168.99.100 netmask 255.255.255.0 broadcast 192.168.99.255 up
```

Se debe tener en cuenta que dirección IP ingresada en el archivo debe ser la misma que la obtenida con el comando docker-machine ip manager. De igual forma la dirección del broadcast debe concordar con la misma red.

* Asignar los permisos adecuados al archivo _bootsync.sh_ y salir del nodo **manager**.

```
$ sudo chmod 755 /var/lib/boot2docker/bootsync.sh
```

```
$ exit
```

* Reiniciar el nodo **manager**.

```
$ docker-machine restart manager
```

* Regenerar el certificado del nodo **manager**.

```
$ docker-machine regenerate-certs manager
```

---

**Inicialización del modo Swarm en el nodo manager.**

* Ingresar al nodo **manager**.

```
$ docker-machine ssh manager
```

* Inicializar el modo Swarm.

```
$ docker swarm init --advertise-addr 192.168.99.100
```

La IP debe colocarse debido a que la máquina virtual creada por docker-machine en VirtualBox contiene varias interfaces de red y varias direcciones IP.  Por tanto, debe especificarle la dirección IP que retorna el comando **docker-machine ip manager**.

La ejecucion del comando nos devuelve el comando necesario para agregar un nodo al Swarm.

Copiar la salida en el portapapeles.

> docker swarm join --token SWMTKN-1-61hpva2ixi24x1dzkfs61y7a5nwuuoq8c2n6onfdowg2knaphv-e8p88sf6r4x0407ek959rvgfj 192.168.99.100:2377

* Salir del nodo **manager**.

```
exit
```

---

**Agregar los  nodos al Swarm**.

Si por algún motivo no ha copiado el comando para incluir un nodo en el Swarm, puede obtenerlo ejecutando el comando **docker swarm join-token worker** dentro del nodo **manager**.

* Ingresar al nodo **worker1**.

```
$ docker-machine ssh worker1
```

* Agregar el nodo **worker1** al Swarm ejecutando el comando que se genero a la salida de la inicialización del Swarm.

```
$ docker swarm join --token SWMTKN-1-61hpva2ixi24x1dzkfs61y7a5nwuuoq8c2n6onfdowg2knaphv-e8p88sf6r4x0407ek959rvgfj 192.168.99.100:2377
```

* Salir del nodo **worker1**.

```
$ exit
```

Repetir estos pasos para los nodos **worker2 **y **worker3**.

---

**Verificar que todos los nodos esten vinculados al Swarm.**

* Ingresar al nodo **manager**.

```
$ docker-machine ssh manager
```

* Listar los nodos del Swarm

```
$ docker node ls

ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
ugr65ekld1jz2zggcqvigrayy *   manager             Ready               Active              Leader
oaiz3i7pf2if28o014mx2xevn     worker1             Ready               Active
d42j6agkb19vz4ccq3hhs8eqs     worker2             Ready               Active
5tg1yrf3kqm6i28rkjso6hfgg     worker3             Ready               Active
```

### CREACIÓN DE LAS IMAGENES EVE Y ANALITICA DE DATOS

Ingresar al nodo **manager** del Swarm sino lo esta.

```
$ docker-machine ssh manager
```

Los archivos Dockerfile que contienen la creación de estas dos imagenes se encuentran en GitHub. Para descargarlos ejecutar:

```
$ git clone https://github.com/alejandrotorresn/Analytic_eve.git
```

El contenido debe ser similar al mostrado a continuación:

```
├── Eve
│   ├── 000-default.conf
│   ├── Dockerfile
│   ├── run.py
│   ├── settings.py
│   ├── user1
│   │   ├── insert.json
│   │   ├── insert.json.gz
│   │   ├── run.py
│   │   └── settings.py
│   └── user2
│       ├── insert.json
│       ├── insert.json.gz
│       ├── run.py
│       └── settings.py
├── miniconda_p27
│   └── Dockerfile
└── README.md
```

Por el momento las imagenes solo seran creadas en el nodo **manger**. Posteriormente se vera que se debe tener la imagen base en donde desee crearse el servicio.

---

**Creación de la imagen de Analitica de datos.**

De aquí en adelante se asume que se encuentra dentro del repositorio descargado de GitHub \(folder **Analytic\_eve**\).

* Ingresar al directorio miniconda\_p27

```
$ cd miniconda_p27
```

* Construir la imagen.

```
$ docker build -t analitica_datos .
```

* Verificar que la imagen ha sido creada correctamente.

```
$ docker images
```

* Verificar que puede ejecutarse el servicio de **Análitica de Datos**.

```
$ docker service create --name test_analitica --constraint 'node.hostname == manager' --publish 8888:8888 analitica_datos
```

* Verificar que el servicio se encuentra ejecutandose.

```
$ docker service ls

ID                  NAME                MODE                REPLICAS            IMAGE                    PORTS
ye8ngif9e4qe        test_analitica      replicated          1/1                 analitica_datos:latest   *:8888->8888/tcp
```

* Ingresar a la dirección [http://192.168.99.100:8888](http://192.168.99.100:8888) para verficar que el servicio es valido.

![](/assets/Analitica_test.png)

* Eliminar el servicio de prueba para el contenedor de análitica.

```
$ docker service rm test_analitica
```

---

**Creación de la Imagen del servicio REST - Eve**

* Ingrese al directorio Eve

```
$ cd Eve
```

* Construir la imagen

```
$ docker build -t eve_apache .
```

Verificar que la imagen ha sido creada correctamente.

```
$ docker images
```

* Para evaluar el servicio de Eve se debe tener el servicio de **MongoDB** en funcionamiento, para ello se ejecuta:

```
$ docker service --name=mongo_eve --constraint 'node.hostname == manager' --publish 27017:27017 mongo
```

* Verificar que puede ejecutarse el servicio de Eve.

```
$ docker service create --name test_eve --constraint 'node.hostname == manager' --publish 80:80 eve_apache
```

* Verificar que el servicio se encuentra ejecutandose.

```
$ docker service ls


ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
546j5e76f8i2        mongo_eve           replicated          1/1                 mongo:latest        *:27017->27017/tcp
elh3v5ugfl5i        test_eve            replicated          1/1                 test_eve:latest     *:80->80/tcp
```

* Ingresar a la dirección [http://192.168.99.100](http://192.168.99.100:8888) para verficar que el servicio es valido.

![](/assets/Eve_test.png)La configuración básica del Servidor REST Eve pide un usuario y una contraseña. Por el momento no se han ingresado usuarios, por lo tanto el servidor Eve mostrara información como la mostrada en la imagen anterior.

* Enviar información a Eve para verificar si esta recibiendo tanto archivos .json como .gzip

## CASO 1.

servicios a correr

1 servicio mongo

1 servicio Eve para el usuario1 - Una sola replica - puerto XX

1 servicio Eve para el usuario2 - Dos replicas - puerto YY

1 servicio de analitica de datos

use case2:

1. full installation

2. mongo + eves running permanently, ocasional analytics container

3. mongo + eve customer1 permanently. eve customer2 + analytics ocasional

4. ganglia/swarm management/query web interface



