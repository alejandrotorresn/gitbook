# CASOS DE USO - SWARM MODE

Para los casos de uso se implementaran un solo nodo **manager **y tres nodos **worker**. Estos nodos se crearan mediante **docker-machine** y en _VirtualBox_.

Se implementaran los servicios de **MongoDB**, un servidor REST basado en **Eve **junto con una configuración personalizada de **Apache** que permite recibir archivos .gzip en el servicio REST y un contedor de **Analítica de Datos** \(Jupyter Notebook y Jupyter Lab\).

## REQUISITOS PREVIOS

1. Creación de las máquinas virtuales en _VirtualBox_
2. Creación del Swarm
3. Creación de las imagenes necesarias \(Eve y Analítica de Datos\)

### CREACIÓN DE LAS MÁQUINAS VIRTUALES

* Creación del nodo Manager \(Es recomendable asignarle un mínimo de 4GB de memoria al nodo manager\):

```
$ docker-machine create -d virtualbox --virtualbox-memory 4096 manager
```

* Creación de los nodos Worker

```
$ docker-machine create -d virtualbox worker1
$ docker-machine create -d virtualbox worker2
$ docker-machine create -d virtualbox worker3
```

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
$ docker-machien ssh manager
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



