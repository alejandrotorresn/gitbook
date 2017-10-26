# SWARM MODE - IMPLEMENTACIÓN EN ANÁLITICA DE DATOS

Para los casos de uso se implementará un solo nodo **manager **y dos nodos **workers** mediante **docker-machine** y en _VirtualBox_.

Se implementaran los servicios de **MongoDB**, Servidores REST basados en **Eve** junto con una configuración personalizada de **Apache** que permite recibir archivos .gzip en el servicio REST y servicios **Analítica de Datos** \(Jupyter Notebook y Jupyter Lab\).

## REQUISITOS PREVIOS

1. [Instalación de Docker Engine](/instalacion.md)
2. [Instalación de Docker Machine](/instalacion-de-docker-machine.md)
3. Creación de las máquinas virtuales en _VirtualBox_ 

**NOTA:** Los dos primeros requisitos pueden satisfacerse siguiendo los enlaces. No es necesario la instalación de _Docker Engine_ si se salta directamente a los nodos mediante **docker-machine ssh node_name** y no se usa **docker-machine env node_name** para conectar el cliente Docker al Docker Engine corriendo en la máquina virtual.

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

 **NOTA:** En la salida de la terminal puede observarse que no se encontro la imagen localmente y se procede a descargarla del repositorio de DockerHub.

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

Antes de crear el Swarm se recomienda establecer la dirección IP del nodo manager como dirección estática, para ello se debe:

* Obtener la dirección dada por el servicio DHCP a la máquina **manager1** creada por docker-machine.

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine ip manager1]
**[warning 192.168.99.100]
```

* Ingresar al nodo **manager1**.

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine ssh manager1]
```

* Crear el archivo **bootsync.sh**

 ```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command sudo vi /var/lib/boot2docker/bootsync.sh]
```

* Ingresar los siguientes comandos en el archivo bootsync.sh

```bash
#!/bin/sh
/etc/init.d/services/dhcp stop
ifconfig eth1 192.168.99.100 netmask 255.255.255.0 broadcast 192.168.99.255 up
```

 Se debe tener en cuenta que la dirección IP ingresada en el archivo debe ser la misma que la obtenida con el comando _docker-machine ip manager_. De igual forma la dirección del broadcast debe concordar con la misma red.

* Asignar los permisos adecuados al archivo _bootsync.sh_ y salir del nodo **manager1**.

 ```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command sudo chmod 755 /var/lib/boot2docker/bootsync.sh]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command exit]
```

* Reiniciar el nodo **manager1**.

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine restart manager1]
```

* Regenerar el certificado del nodo **manager1**.

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine regenerate-certs manager1]
```

---

**Inicialización del modo Swarm en el nodo manager**

---


* Ingresar al nodo **manager1**.

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine ssh manager1]
```

* Inicializar el modo Swarm y salir del nodo.

 ```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker swarm init --advertise-addr 192.168.99.100]
Swarm initialized: current node (0ayliiwjtgo2i4i4npsw4kj0k) is now a manager.
To add a worker to this swarm, run the following command:
**[warning      docker swarm join --token SWMTKN-1-0e9r5688ui3q2hdkm7xzal4o83bktaeiuo8jetljp4z0povphj-9era17h2lj5493xt4knb38o7t 192.168.99.100:2377]
To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command exit]
```

 La IP debe colocarse debido a que la máquina virtual creada por docker-machine en VirtualBox contiene varias interfaces de red y varias direcciones IP.  Esta IP es la retornada por el comando **docker-machine ip manager**.

 La ejecucion del comando nos devuelve el comando necesario para agregar un nodo al Swarm.

 Copiar la salida en el portapapeles.

 > docker swarm join --token SWMTKN-1-0e9r5688ui3q2hdkm7xzal4o83bktaeiuo8jetljp4z0povphj-9era17h2lj5493xt4knb38o7t 192.168.99.100:2377

---
**Agregar los nodos al Swarm**

---

Si por algún motivo no ha copiado el comando para incluir un nodo en el Swarm, puede obtenerlo ejecutando el comando **docker swarm join-token worker** dentro del nodo **manager1**.

* Ingresar al nodo **worker1**.

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine ssh worker1]
```

* Agregar el nodo **worker1** al Swarm ejecutando el comando que se genero a la salida de la inicialización del Swarm y salir del nodo.

 ```
**[terminal]
**[prompt docker@worker1]**[path ~]**[delimiter $ ]**[command docker swarm join --token SWMTKN-1-0e9r5688ui3q2hdkm7xzal4o83bktaeiuo8jetljp4z0povphj-9era17h2lj5493xt4knb38o7t 192.168.99.100:2377]
**[prompt docker@worker1]**[path ~]**[delimiter $ ]**[command exit]
```

Repetir estos pasos para el nodo **worker2**.

---
**Verificar que todos los nodos esten vinculados al Swarm**

---

* Ingresar al nodo **manager1**.

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine ssh manager1]
```

* Listar los nodos del Swarm

 ```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker node ls]
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
2mlcvbq1jlvkzygogoxym86o2 *   manager1            Ready               Active              Leader
9vzt9zkvye2na0udnvvt8k29d     worker1             Ready               Active              
r9g23qn715xuemvvielc4nr8k     worker2             Ready               Active              
```

## CONFIGURACIÓN DEL ENTORNO DE ANÁLITICA DE DATOS


Ingresar al nodo **manager1** del Swarm sino lo esta.

```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine ssh manager1]
```

Los archivos Dockerfile que contienen la creación de estas dos imagenes se encuentran en GitHub. Para descargarlos ejecutar:

```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command git clone https://github.com/alejandrotorresn/Analytic_eve.git]
```

El contenido debe ser similar al mostrado a continuación:

```bash
├── Analitica
│   └── Dockerfile
├── Customers
│   ├── customer1
│   │   ├── insert.json
│   │   ├── insert.json.gz
│   │   ├── run.py
│   │   └── settings.py
│   └── customer2
│       ├── insert.json
│       ├── insert.json.gz
│       ├── run.py
│       └── settings.py
├── Eve
│   ├── 000-default.conf
│   └── Dockerfile
└── README.md
```

---
**Creación de la Imagen de Analítica de Datos**

---

De aquí en adelante se asume que se encuentra dentro del repositorio descargado de GitHub \(directorio **Analytic_eve**).

* Ingresar al directorio Analitica

 ```
**[terminal]
**[prompt docker@manager1]**[path ~/Analytic_eve]**[delimiter $ ]**[command cd Analitica]
```

* Construir la imagen.

 ```
**[terminal]
**[prompt docker@manager1]**[path ~/Analytic_eve/Analitica]**[delimiter $ ]**[command docker build -t analitica_datos .]
Sending build context to Docker daemon  3.584kB
Step 1/30 : from ubuntu
 ---> 747cb2d60bbe
 ...
 Step 30/30 : CMD ["jupyter-lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--NotebookApp.token="]
 ---> Running in 7269b31a62e5
 ---> db974366bc12
Removing intermediate container 242aaae16bcf
Successfully built db974366bc12
Successfully tagged analitica_datos:latest
```

* Verificar que la imagen ha sido creada correctamente.

 ```
**[terminal]
**[prompt docker@manager1]**[path ~/Analytic_eve/Analitica]**[delimiter $ ]**[command docker images]
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
analitica_datos     latest              db974366bc12        About a minute ago   3.19GB
ubuntu              latest              747cb2d60bbe        2 weeks ago          122MB
```

---
**Creación de la Imagen del Servicio REST - Eve**

---

* Ingrese al directorio Eve

 ```
**[terminal]
**[prompt docker@manager1]**[path ~/Analytic_eve]**[delimiter $ ]**[command cd Eve]
```

* Construir la imagen

 ```
**[terminal]
**[prompt docker@manager1]**[path ~/Analytic_eve/Eve]**[delimiter $ ]**[command docker build -t eve_apache .]
Sending build context to Docker daemon  5.632kB
Step 1/29 : FROM ubuntu
 ---> 747cb2d60bbe
 ...
Step 28/29 : RUN echo "/usr/sbin/apache2 -D FOREGROUND" >> start.sh
 ---> Running in b759cd0878e0
 ---> ce4d17330893
Step 29/29 : CMD ["sh","start.sh"]
 ---> Running in 15d117d66f5b
 ---> a3156b948c5e
Removing intermediate container 2bace4da73c3
Successfully built a3156b948c5e
Successfully tagged eve_apache:latest 
```

* Verificar que la imagen ha sido creada correctamente.

```
**[terminal]
**[prompt docker@manager1]**[path ~/Analytic_eve/Eve]**[delimiter $ ]**[command docker images]
```



---
**Descarga de la Imagen de MongoDB**

---



---
**Creación del Repositorio Local de Imagenes**

---







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



## CASOS DE USO




### CASO DE USO BÁSICO - DESPLIEGUE DE SERVICIOS DE ANÁLITICA


### CASOS DE USO - AVANZADO









### CREACIÓN DE LAS IMAGENES EVE Y ANALITICA DE DATOS


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



