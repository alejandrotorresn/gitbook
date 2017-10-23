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



