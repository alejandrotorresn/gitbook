# INSTALACION DE DOCKER EN SWARM

La instalación de Docker en modo Swarm se hará en:

* VirtualBox
* AWS

## VIRTUALBOX

### REQUISITOS PREVIOS

Creación de las máquinas virtuales para la configuración del Swarm. Para esta guía se instalaran dos nodos; uno de ellos servirá como manager/worker y el otro solo como worker.

#### Creación de los nodos con docker-machine

El comando docker-machine crea las máquinas virtuales con el docker engine ya instalado.

```
$ docker-machine create -d virtualbox manager1
$ docker-machine create -d virtualbox worker1
```





Al crear el nodo manager1 se asigna una dirección IP a través del servicio DHCP proporcionado por docker-machine. Se recomienda que este nodo tenga una dirección estática con el fin de que al reiniciar las máquinas no se pierda la conexión entre el manager y los workers.



Para colocar una IP estática en una máquina creada por docker-machine en virtualbox se sigue el siguiente procedimiento:



    Obtener la IP asignada por docker-machine:



$ docker-machine ip manager1





192.168.99.100



    Ingresar a la máquina manager1:



$ docker-machine ssh manager1



    Crear el archivo bootsync.sh



$ sudo vi /var/lib/boot2docker/bootsync.sh



    Ingresar el siguiente contenido en donde la dirección IP es la obtenida mediante el comando docker-machine ip manager1. Se debe tener en cuenta que la dirección del broadcast debe corresponder a la misma red de la IP.



    \#!/bin/sh

    /etc/init.d/services/dhcp stop

    ifconfig eth1 192.168.99.100 netmask 255.255.255.0 broadcast 192.168.99.255 up



    Asignarle los permisos adecuados al archivo bootsync.sh y salir de la máquina virtual:



$ sudo chmod 755 /var/lib/boot2docker/bootsync.sh

$ exit



    Reiniciar la máquina manager1:



$ docker-machine restart manager1



    Regenerar el certificado de la máquina manager1:



exit



$ docker-machine regenerate-certs manager1





CREAR UN SWARM



Ingresar al nodo manager1:



$ docker-machien ssh manager1



Inicializar el modo Swarm:



$ docker swarm init --advertise-addr 192.168.99.100





La ejecucion del comando nos devuelve el comando necesario para agregar un nodo al Swarm



docker swarm join --token SWMTKN-1-61hpva2ixi24x1dzkfs61y7a5nwuuoq8c2n6onfdowg2knaphv-e8p88sf6r4x0407ek959rvgfj 192.168.99.100:2377



AGREGAR NODOS A SWARM

UN NODO WORKER





Para obtener el comando para incluir un worker al Swarm:



    Ingresar al nodo manager \(manager1\):



$ docker-machine ssh manager1



    Ejecutar el comando docker swarm join-token worker y salir de la máquina manager1:



$ docker swarm join-token worker





To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-61hpva2ixi24x1dzkfs61y7a5nwuuoq8c2n6onfdowg2knaphv-e8p88sf6r4x0407ek959

rvgfj 192.168.99.100:2377





$ exit



Comando se usara para agregar un worker.



    docker swarm join --token SWMTKN-1-61hpva2ixi24x1dzkfs61y7a5nwuuoq8c2n6onfdowg2knaphv-e8p88sf6r4x0407ek959rvgfj 192.168.99.100:2377



NOTA: Para cada configuración se genera un token personalizado.





Para obtener el comando para incluir un worker al Swarm:



    Ingresar al nodo manager \(manager1\):



$ docker-machine ssh manager1



    Ejecutar el comando docker swarm join-token worker y salir de la máquina manager1:



$ docker swarm join-token worker





To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-61hpva2ixi24x1dzkfs61y7a5nwuuoq8c2n6onfdowg2knaphv-e8p88sf6r4x0407ek959

rvgfj 192.168.99.100:2377





$ exit



Comando se usara para agregar un worker.



    docker swarm join --token SWMTKN-1-61hpva2ixi24x1dzkfs61y7a5nwuuoq8c2n6onfdowg2knaphv-e8p88sf6r4x0407ek959rvgfj 192.168.99.100:2377



NOTA: Para cada configuración se genera un token personalizado.





AWS



