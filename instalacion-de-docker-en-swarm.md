# INSTALACION DE DOCKER EN SWARM

![](/assets/swarm-diagram.png)

La instalación de Docker en modo Swarm se hará en:

* VirtualBox
* AWS

## VIRTUALBOX

### REQUISITOS PREVIOS

Creación de las máquinas virtuales para la configuración del Swarm. Para esta guía se instalaran dos nodos; uno de ellos servirá como manager/worker y el otro solo como worker.

#### Creación de los nodos con docker-machine

El comando docker-machine crea las máquinas virtuales con el docker engine ya instalado.


```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter  $ ]**[command docker-machine create -d virtualbox manager1]

Running pre-create checks...
Creating machine...
...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env manager1

**[prompt user@server]**[path ~]**[delimiter  $ ]**[command docker-machine create -d virtualbox worker1]
Running pre-create checks...
Creating machine...
...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env worker1
```

**Nota:** En los casos de uso se usa el nodo manager1 con un mínimo de 4GB de memoria. Si va a implementar los casos de uso se recomienda que cree la máquina virtual con esta configuración, mediante el siguiente comando:

  ```
  **[terminal]
  **[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine rm manager1]
  **[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine create -d virtualbox --virtualbox-memory 4096 manager1]
  ```

Al crear el nodo **manager1** se asigna una dirección IP a través del servicio DHCP. Se recomienda que este nodo tenga una dirección estática con el fin de que al reiniciar las máquinas no se pierda la conexión entre el manager y los workers.

##### IP estática en una máquina creada por _docker-machine_ en Virtualbox:

* Obtener la IP asignada por _docker-machine_:

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter  $ ]**[command docker-machine ip manager1]
192.168.99.100
```

* Ingresar al nodo **manager1**:

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter  $ ]**[command docker-machine ssh manager1]
```

* Crear el archivo bootsync.sh

 ```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter  $ ]**[command sudo vi /var/lib/boot2docker/bootsync.sh]
```

* Ingresar el siguiente contenido en donde la dirección IP es la obtenida mediante el comando **docker-machine ip manager1**. Se debe tener en cuenta que la dirección del broadcast debe corresponder a la misma red de la IP.

 ```bash
#!/bin/sh
/etc/init.d/services/dhcp stop
ifconfig eth1 192.168.99.100 netmask 255.255.255.0 broadcast 192.168.99.255 up
```

* Asignarle los permisos adecuados al archivo _bootsync.sh_ y salir de la máquina virtual:

 ```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter  $ ]**[command sudo chmod 755 /var/lib/boot2docker/bootsync.sh]
**[prompt docker@manager1]**[path ~]**[delimiter  $ ]**[command exit]
```

* Reiniciar el nodo **manager1**:

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter  $ ]**[command docker-machine restart manager1]
```

* Regenerar el certificado del nodo **manager1**:

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter  $ ]**[command docker-machine regenerate-certs manager1]
```

### CREAR UN SWARM

* Ingresar al nodo **manager1**:

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter  $ ]**[command docker-machine ssh manager1]
```

* Inicializar el modo Swarm y salir del nodo **manager1**:

 ```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter  $ ]**[command docker swarm init --advertise-addr 192.168.99.100]
Swarm initialized: current node (0ayliiwjtgo2i4i4npsw4kj0k) is now a manager.
To add a worker to this swarm, run the following command:
**[warning      docker swarm join --token SWMTKN-1-0e9r5688ui3q2hdkm7xzal4o83bktaeiuo8jetljp4z0povphj-9era17h2lj5493xt4knb38o7t 192.168.99.100:2377]
To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
**[prompt user@server]**[path ~]**[delimiter $ ]**[command exit]
```

La ejecucion del comando nos devuelve el comando necesario para agregar un nodo al Swarm

> docker swarm join --token SWMTKN-1-0e9r5688ui3q2hdkm7xzal4o83bktaeiuo8jetljp4z0povphj-9era17h2lj5493xt4knb38o7t 192.168.99.100:2377

### AGREGAR NODOS A SWARM

#### UN NODO WORKER

Para obtener el comando para incluir un worker al Swarm:

* Ingresar al nodo manager \(**manager1**\):

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter  $ ]**[command docker-machine ssh manager1]
```

* Ejecutar el comando docker swarm join-token worker:

 ```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter  $ ]**[command docker swarm join-token worker]
**[warning      docker swarm join --token SWMTKN-1-0e9r5688ui3q2hdkm7xzal4o83bktaeiuo8jetljp4z0povphj-9era17h2lj5493xt4knb38o7t 192.168.99.100:2377]
```

 **NOTA**: Para cada configuración se genera un token personalizado.

* Salir de la máquina **manager1**:

 ```
**[terminal]
**[prompt user@manager1]**[path ~]**[delimiter  $ ]**[command exit]
```

* Ingresar al nodo worker \(**worker1**\):

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter  $ ]**[command docker-machine ssh worker1]
```

* Agregar el nodo **worker1** al Swarm mediante la ejecución del comando retornado al inicializar el Swarm y salir de la máquina:

 ```
**[terminal]
**[prompt docker@worker1]**[path ~]**[delimiter  $ ]**[command docker swarm join --token SWMTKN-1-0e9r5688ui3q2hdkm7xzal4o83bktaeiuo8jetljp4z0povphj-9era17h2lj5493xt4knb38o7t 192.168.99.100:2377]
**[warning      This node joined a swarm as a worker.]
**[prompt docker@worker1]**[path ~]**[delimiter  $ ]**[command exit]
```

* Verificar que los nodos estes vinculados al Swarm

 * Ingresar al nodo **manager1**
  ```
 **[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine ssh manager1]
```

 * Listar los nodos del Swarm:
 ```
 **[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker node ls]
```

## AWS



