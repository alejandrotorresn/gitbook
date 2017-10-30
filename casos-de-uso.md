# SWARM MODE - IMPLEMENTACIÓN EN ANÁLITICA DE DATOS

Para los casos de uso se implementará un solo nodo **manager **y dos nodos **workers** mediante **docker-machine** y en _VirtualBox_.

Se implementaran los servicios de **MongoDB**, Servidores REST basados en **Eve** junto con una configuración personalizada de **Apache** que permite recibir archivos .gzip en el servicio REST y servicios **Analítica de Datos** \(Jupyter Notebook y Jupyter Lab\).

## REQUISITOS PREVIOS

1. [Instalación de Docker Engine](/instalacion.md)
2. [Instalación de Docker Machine](/instalacion-de-docker-machine.md)
3. Creación de las máquinas virtuales en _VirtualBox_ 

**NOTA:** Los dos primeros requisitos pueden satisfacerse siguiendo los enlaces. No es necesario la instalación de _Docker Engine_ si se salta directamente a los nodos mediante **docker-machine ssh node\_name** y no se usa **docker-machine env node\_name** para conectar el cliente Docker al Docker Engine corriendo en la máquina virtual.

## CREACIÓN DE LAS MÁQUINAS VIRTUALES

Docker-machine usa la imagen **boot2docker** para crear las máquinas virtuales. Esta imagen instala un sistema operativo con un sistema de archivos de solo lectura y al reiniciar un nodo, todos los datos almacenados en este se perderán. Para asegurar la persistencia de los datos se hace necesario la creación de [volumenes](https://docs.docker.com/engine/admin/volumes/volumes/).

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

## MONITOREO DE DOCKER SWARM

* Descargar el archivo _docker-stack.yml_

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command wget https://raw.githubusercontent.com/botleg/swarm-monitoring/master/docker-stack.yml]
  ```

* Desplegar los servicios de monitoreo:

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker stack deploy -c docker-stack.yml monitor]
  ```

  **NOTA:** Debe esperar un tiempo hasta que los servicios se inicien antes de poder crear la base de datos del siguiente paso.

* Crear la base de datos llamada **cadvisor** en InfluxDB:
 ```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker exec `docker ps | grep -i influx | awk '{print $1}'` influx -execute 'CREATE DATABASE cadvisor']   
```

  Si recibe algún mensaje informando que no se encuentra el contenedor _influx_ es debido a que aun no ha sido creado por el servicio.

* Abrir grafana en el navegador (http://192.168.99.100). El servicio se ha habilitado en el puerto 80 y el usuario es **admin** al igual que la contraseña. Cabe hacer notar que puede usarse cualquier IP vinculada a un nodo del Swarm.

![](/assets/grafana.png)

* En Grafana crear un nuevo _data source_ con el nombre **influx**, de tipo **InfluxDB**, con url [http://influx:8086](http://influx:8086) y database **cadvisor**.

![](/assets/influxdb_grafana.png)

* Descargar el archivo dashboard.json en el servidor base.

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command exit]
  **[prompt user@server]**[path ~]**[delimiter $ ]**[command wget https://raw.githubusercontent.com/botleg/swarm-monitoring/master/dashboard.json]
  ```

* En el la ventada del navegador donde se encuentra abierto el servicio de Grafana, hacer clic en el icono de la parte superior izquierda, seleccionar _Dashboards_ y luego _import_. Cargar el archivo _dashboard.json_.

![](/assets/dashboard_grafana.png)

* Darle un nombre al Dashboard y seleccionar la fuente de datos Influx.

![](/assets/dashboard_import_grafana.png)

* Finalmente el servicio de monitoreo quedará como se muestra en la siguiente imagen:

![](/assets/Monitoring.png)

## CONFIGURACIÓN DEL ENTORNO DE ANÁLITICA DE DATOS

Ingresar al nodo **manager1** del Swarm si no lo está.

```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine ssh manager1]
```

Los archivos Dockerfile que contienen la creación de estas dos imagenes se encuentran en GitHub. Para descargarlos ejecutar:

```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command git clone https://github.com/alejandrotorresn/Analytic_eve.git]
```

La estructura debe ser similar a la mostrada a continuación:

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

De aquí en adelante se asume que se encuentra dentro del repositorio descargado de GitHub \(directorio **Analytic\_eve**\).

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

```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker image pull mongo]
Using default tag: latest
latest: Pulling from library/mongo
...
Digest: sha256:39a5ddbed305b7a0d90e1c8c777f43680bafeb982135f775c22fb0f901b67f98
Status: Downloaded newer image for mongo:latest
```

---

**Creación del Repositorio Local de Imagenes**

---

Para la creación del repositorio local se usará [_Docker Registry_](https://docs.docker.com/registry/); es un servidor _stateless_ que almacena y permite distribuir imagenes de Docker. En otras palabras, permite crear un repositorio local de imagenes que todos pueden acceder sin necesidad de construir una nueva imagen o descargarla de DockerHub.

* Crear el volumen para almacenar las imagenes. El servicio de _registry_ se ejecutara en el nodo **manager1** por tanto el volumen se creará en este nodo.

 ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker volume create --name registry]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker volume ls]
  DRIVER VOLUME NAME
  local registry
```

* Ejecutar el servicio.

 ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker service create --name registry --constraint 'node.hostname == manager1'  --publish 5000:5000 --mount type=volume,source=registry,target=/var/lib/registry registry:2]
  j0961u4mqk6krwur9f2x4qo2n
  overall progress: 1 out of 1 tasks 
  1/1: running   [==================================================>] 
  verify: Service converged
```

* Listar las imagenes.

 ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker image ls]
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  eve_apache          latest              a3156b948c5e        2 hours ago         529MB
  analitica_datos     latest              db974366bc12        4 hours ago         3.19GB
  mongo               latest              a28fdc58a538        19 hours ago        361MB
  registry            <none>              2ba7189700c8        22 hours ago        33.3MB
  ubuntu              latest              747cb2d60bbe        2 weeks ago         122MB
```

* Etiquetar las imagenes con una etiqueta adicional donde se especifique el hostname y el puerto. Docker automáticamente interpreta que esta es la ubicación del _registry_.

  Para el repositorio de Análitica solo se etiquetaran las imagenes de eve\_apache, analitica\_datos y mongo.

 ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker tag eve_apache localhost:5000/eve_apache]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker tag analitica_datos localhost:5000/analitica_datos]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker tag mongo localhost:5000/mongo]
```

* Listar las imagenes para ver que se han reetiquetado correctamente.

 ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker images]
  REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
  eve_apache                       latest              a3156b948c5e        3 hours ago         529MB
  localhost:5000/eve_apache        latest              a3156b948c5e        3 hours ago         529MB
  analitica_datos                  latest              db974366bc12        4 hours ago         3.19GB
  localhost:5000/analitica_datos   latest              db974366bc12        4 hours ago         3.19GB
  mongo                            latest              a28fdc58a538        20 hours ago        361MB
  localhost:5000/mongo             latest              a28fdc58a538        20 hours ago        361MB
  registry                         <none>              2ba7189700c8        22 hours ago        33.3MB
  ubuntu                           latest              747cb2d60bbe        2 weeks ago         122MB
```

* Subir las imagenes al _registry_ local que se encuentra corriendo en localhost:5000.

 ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker push localhost:5000/eve_apache]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker push localhost:5000/analitica_datos]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker push localhost:5000/mongo]
```

* Asegurarse que las imagenes han sido cargadas al repositorio local.

 ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command curl localhost:5000/v2/_catalog]
  **[warning {"repositories":["analitica_datos","eve_apache","mongo"]}]
```

* Puede remover las imagenes locales sin afectar las imagenes que ya se han cagador al _registry_.

 ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker image remove eve_apache]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker image remove localhost:5000/eve_apache]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker image remove analitica_datos]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker image remove localhost:5000/analitica_datos]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker image remove mongo]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker image remove localhost:5000/mongo]
```

---

**Creación de la Red Overlay para los Servicios**

---

La red Overlay permite a los servicios comunicarse entre ellos. Con esto los servicios no necesitan conocer la IP ni el nombre del nodo en donde se encuentra alojado el contenedor que los esta ejecutando. En otras palabras, los servicios puede comunicarse gracias a un servicio de DNS de Docker en modo Swarm mediante una red Overlay.

```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker network create -d overlay services_overlay]
```

Para ver la nueva red creada, ejecutar:

```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker network ls]
NETWORK ID          NAME                DRIVER              SCOPE
28d8d60ce1a8        bridge              bridge              local
2ea2660bf2f8        docker_gwbridge     bridge              local
672e15f6fe15        host                host                local
tjjxkttc4joh        ingress             overlay             swarm
92334a23b2e2        none                null                local
**[warning 0i28klyx8qiq        services_overlay    overlay             swarm]
```

## CASOS DE USO

Los casos de uso se dividen en dos en donde la primera parte abordará el despliegue básico de los servicios de MongoDB, Eve y Analítica de datos. En la segunda parte se desplegarán los servicios en situaciones particulares con el fin de mostrar la versatilidad de los servicios.

### CASO DE USO BÁSICO - DESPLIEGUE DE SERVICIOS DE ANÁLITICA

Para este caso de uso se despliegan los servicios de MongoDB, Eve y Analítica de Datos para un solo _Customer_ y en un único nodo \(**manager1**\). Cabe recordar que la infraestructura creada para estos casos de uso cuenta con un nodo **manager** y dos nodos **workers**, por lo tanto, en el comando de la creación de los servicios se especificará el nodo donde se desplegarán los servicios. Si la infraestructura solo contara con un solo nodo \(nodo **manager**\) no sería necesario esta especificación \(los nodos **manager** tambien funcionan con nodos **workers**, al menos que se especifique lo contrario\).

Para desplegar el primer caso de uso, se debe:

Ingresar al nodo **manager1** del Swarm sino lo esta.

```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine ssh manager1]
```

---

**Despliegue del servicio de MongoDB**

---

* [Creación de los volumenes en Docker](https://docs.docker.com/engine/admin/volumes/volumes/) para el almacenamiento de los datos y archivos de configuración. Se debe tener en cuenta que la creación de los volumenes se hace solo en el nodo donde se ejecutan los comandos. De forma nativa Docker no comparte estos volumenes ni su contenido, por lo tanto, los volumenes se deben crear en el nodo en donde se desea ejecutar el servicio de MongoDB y si se desea sincronizar su contenido se dede implementar algún servicio como NFS, GlusterNF, etc.

  Para este caso de uso se implementa MongoDB en un solo nodo y sus volumenes se crearán en el nodo donde se despliega el servicio.

 ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker volume create --name mongodata]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker volume create --name mongoconfig]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker volume ls]
  DRIVER              VOLUME NAME
  local               mongoconfig
  local               mongodata
```

* Ejecutar el servicio de MongoDB.

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker service create --name mongo_eve  --replicas 1 --network services_overlay --publish 27017:27017 --mount type=volume,source=mongodata,target=/data/db --mount type=volume,source=mongoconfig,target=/data/configdb --constraint 'node.hostname == manager1' localhost:5000/mongo]
  ```

  ** Descripción de los parámetros del servicio de Mongo**

  * --name: Nombre del Servicio
  * --replicas: Número de espejos o replicas del servicio
  * --network: Nombre de la red a la que se enlazará el servicio. Para este caso se usa una red overlay con el fin de comunicar servicios
  * --publish: Puerto a mapear. \(Número de puerto del Swarm: Número de puerto del servicio\)
  * --mount: Folders o Volumenes a montar dentro del contenedor que ofrece el servicio
  * --constraint: Especifica el nombre de la máquina en la cual se desplegará el servicio  

* Verificar que el servicio de MongoDB \(mongo\_eve\) se encuentre en ejecución.

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker service ls]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker service inspect --pretty mongo_eve]
  ```

---

**Despliegue del servicio de Eve**

---

* Descargar los archivos de ejemplo desde el repositorio de GitHub si no los tiene en el nodo **manager1**. Se debe recordar que los nodos estan construidos en base a la imagen **boot2docker** que solo crea contendores con sistemas de archivos de solo lectura y al apagar o reiniciar los nodos la información allí almacenada se perderá.

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command git clone https://github.com/alejandrotorresn/Analytic_eve.git]
  ```

* Ingresar a la carpeta Analytic\_eve/Customers.

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command cd Analytic_eve/Customers]
  **[prompt docker@manager1]**[path ~/Analytic_eve/Customers]**[delimiter $ ]**[command ls]
  **[warning customer1/ customer2/]
  ```

* Dentro se encuentran dos carpetas con los archivos de configuración para levantar el servicio del servidor REST construido con Eve \(run.py y settings.py\). Además, se encuentran otros dos archivos con contenido de ejemplo para insertar en la base de datos usando el servidor Eve \(insert.json y insert.json.gz\).

* Ingresar a la carpeta customer1.

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~/Analytic_eve/Customers]**[delimiter $ ]**[command cd customer1]
  ```

* Lanzar el servicio de Eve:

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~/Analytic_eve/Customers/customer1]**[delimiter $ ]**[command docker service create --name eve_customer1 --replicas 1 --network services_overlay --publish 6001:80 --env MONGO_HOST=mongo_eve --env MONGO_PORT=27017 --env MONGO_DBNAME=customer1_db --mount type=bind,source=/home/docker/Analytic_eve/Customers/customer1,destination=/home/eve  --constraint 'node.hostname == manager1' localhost:5000/eve_apache]
  ```

  ** Descripción de los parámetros del servicio de Eve**

  * --name: Nombre del servicio
  * --replicas: Número de espejos o replicas del servicio
  * --network: Nombre de la red a la que se enlazará el servicio. Para este caso se usa una red overlay con el fin de comunicar servicios
  * --publish: Puerto a mapear. \(Número de puerto del Swarm: Número de puerto del servicio\) 
  * --env: El servicio de Eve requiere de tres variables de ambiente; MONGO\_HOST: Nombre del servicio de MongoDB, MONGO\_PORT: Puerto de Mongo \(Puerto 27017 por defecto\) y MONGO\_DBNAME: Nombre de la base de datos del customer1.
  * --mount: Folders o Volumenes a montar dentro del contenedor que ofrece el servicio. Para el caso de Eve se monta el directorio que contiene los archivos _run.py_ y _setting.py_ necesarios para inicializar el servicio.
  * --constraint: Especifica el nombre de la máquina en la cual se desplegará el servicio

* Verificar que el servidor Eve se encuentre en ejecución.

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~/Analytic_eve/Customers/customer1]**[delimiter $ ]**[command curl http://192.168.99.100:6001]
  **[warning {"_links": {"child": [{"href": "user", "title": "user"}]}}]
  ```

  La respuesta a la petición muestra que existe un _child_ **user** al cual podemos ingresar a consultar información, pero por el momento la Base de datos _customer1\_db_ no contiene ninguna. La razón de ello es porque aún no se ha hecho la primera carga de datos. Si se consulta este podra verse la siguiente respuesta:

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~/Analytic_eve/Customers/customer1]**[delimiter $ ]**[command curl http://192.168.99.100:6001/user]
  **[warning {"_error": {"code": 401, "message": "Please provide proper credentials"}, "_status": "ERR"}]
  ```

  Por el momento no existen usuarios y no puede proporcionarse las credenciales necesarias para consultar la información.

  Dentro de la carpeta **customer1** se encuentran dos archivos a modo de ejemplo. El primero de ellos es _insert.json_ que contiene la información mostrada a continuación:

  ```json
  [{"username":"rgomez", "password":"456", "phone":"9998765"}]
  ```

  Para realizar el envío de estos datos se debe ejecutar:

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~/Analytic_eve/Customers/customer1]**[delimiter $ ]**[command curl -u admin1:admin1  -v -s -d @insert.json -H "Content-Type: application/json" -X POST http://192.168.99.100:6001/user]
   Trying 192.168.99.100...
  Connected to 192.168.99.100 (192.168.99.100) port 6001 (#0)
  Server auth using Basic with user 'admin1'
  POST /user HTTP/1.1
  Host: 192.168.99.100:6001
  Authorization: Basic YWRtaW4xOmFkbWluMQ==
  User-Agent: curl/7.49.1
  Accept: */*
  Content-Type: application/json
  Content-Length: 60

  upload completely sent off: 60 out of 60 bytes
  HTTP/1.1 201 CREATED
  Date: Sat, 28 Oct 2017 01:27:28 GMT
  Server: Eve/0.7.4 Werkzeug/0.11.15 Python/3.5.2
  Content-Type: application/json
  Content-Length: 275
  Location: http://192.168.99.100:6001/user/59f3dd0019e7de0005a1e238
  Connection #0 to host 192.168.99.100 left intact 
  **[warning {"_created": "Sat, 28 Oct 2017 01:27:28 GMT", "_updated": "Sat, 28 Oct 2017 01:27:28 GMT", "_status": "OK", "_id": "59f3dd0019e7de0005a1e238", "_etag": "3a3e866d706a9f793fc1f16224f1fe5252f7b9b7", "_links": {"self": {"href": "user/59f3dd0019e7de0005a1e238", "title": "User"}}}]
  ```

  Puede observarse que el estatus del envio es **OK**.

  **NOTA:** Para este ejemplo se usa un usuario administrador que permite el envío de información a la base de datos \(**user:**admin1, **password:**admin1\). Este usuario se encuentra especifícado dentro del archivo _run.py_.

  Eve verifica la existencia de la base de datos en el servidor mongo \(**mongo\_eve**\) para realizar el envío de la información, si esta no existe, Eve la crea de manera automática y prosigue con la transferencia de datos.

  La información que se ha enviado a la base de datos crea un usuario que permite realizar consultas de la información disponible a través del servidor Eve.

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~/Analytic_eve/Customers/customer1]**[delimiter $ ]**[command curl -u rgomez:456 http://192.168.99.100:6001/user]
  {"_meta": {"max_results": 25, "page": 1, "total": 1}, "_links": {"self": {"href": "user", "title": "user"}, "parent": {"href": "/", "title": "home"}}, "_items": [{"_created": "Sat, 28 Oct 2017 01:27:28 GMT", "_updated": "Sat, 28 Oct 2017 01:27:28 GMT", "password": "456", "phone": "9998765", "_id": "59f3dd0019e7de0005a1e238", "_etag": "3a3e866d706a9f793fc1f16224f1fe5252f7b9b7", "_links": {"self": {"href": "user/59f3dd0019e7de0005a1e238", "title": "User"}}, "username": "rgomez"}]}
  ```

  El servidor Eve esta configurado para aceptar envios de información en archivos json comprimidos como gzip. Dentro de los archivos de ejemplo se ecnuentra el archivo insert.json.gzip para el cual se ejecutará el siguiente comando:

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~/Analytic_eve/Customers/customer1]**[delimiter $ ]**[command curl -u admin1:admin1 -v -s --trace-ascii http_trace.log --data-binary @insert.json.gz -H "Content-Type: application/json" -H "Content-Encoding: gzip" -X POST http://192.168.99.100/user]
   Trying 192.168.99.100...
  Connected to 192.168.99.100 (192.168.99.100) port 6001 (#0)
  Server auth using Basic with user 'admin1'
  POST /user HTTP/1.1
  Host: 192.168.99.100:6001
  Authorization: Basic YWRtaW4xOmFkbWluMQ==
  User-Agent: curl/7.49.1
  Accept: */*
  Content-Type: application/json
  Content-Encoding: gzip
  Content-Length: 87

  upload completely sent off: 87 out of 87 bytes
  HTTP/1.1 201 CREATED
  Date: Sun, 29 Oct 2017 22:43:01 GMT
  Server: Eve/0.7.4 Werkzeug/0.11.15 Python/3.5.2
  Content-Type: application/json
  Content-Length: 275
  Location: http://192.168.99.100:6001/user/59f65975b18b1b0005660c9f

  Connection #0 to host 192.168.99.100 left intact
  **[warning {"_etag": "0b995ae393b45be1b6cf8025f2aeb26e30a0e9ee", "_id": "59f65975b18b1b0005660c9f", "_updated": "Sun, 29 Oct 2017 22:43:01 GMT", "_status": "OK", "_created": "Sun, 29 Oct 2017 22:43:01 GMT", "_links": {"self": {"title": "User", "href": "user/59f65975b18b1b0005660c9f"}}}]
  ```

  Puede observarse que el estatus del envio es **OK**.

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~/Analytic_eve/Customers/customer1]**[delimiter $ ]**[command curl -u rgomez:456 http://192.168.99.100:6001/user]
  {"_meta": {"max_results": 25, "page": 1, "total": 1}, "_links": {"self": {"href": "user", "title": "user"}, "parent": {"href": "/", "title": "home"}}, "_items": [{"_created": "Sat, 28 Oct 2017 01:27:28 GMT", "_updated": "Sat, 28 Oct 2017 01:27:28 GMT", "password": "456", "phone": "9998765", "_id": "59f3dd0019e7de0005a1e238", "_etag": "3a3e866d706a9f793fc1f16224f1fe5252f7b9b7", "_links": {"self": {"href": "user/59f3dd0019e7de0005a1e238", "title": "User"}}, "username": "rgomez"}]}
  ```

  El servidor Eve esta configurado para aceptar envios de información en archivos json comprimidos como gzip. Dentro de los archivos de ejemplo se ecnuentra el archivo insert.json.gzip que contiene un nuevo usuario llamado _ltorres_. Para insertar la información del nuevo usuario en la base de datos se ejecuta el siguiente comando:

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~/Analytic_eve/Customers/customer1]**[delimiter $ ]**[command curl -u rgomez:456 http://192.168.99.100:6001/user]
  {"_links": {"parent": {"title": "home", "href": "/"}, "self": {"title": "user", "href": "user"}}, "_meta": {"max_results": 25, "page": 1, "total": 2}, "_items": [
  {"username": "rgomez", "password": "456", "phone": "9998765", "_etag": "3a3e866d706a9f793fc1f16224f1fe5252f7b9b7", "_id": "59f3dd0019e7de0005a1e238", "_updated": "Sat, 28 Oct 2017 01:27:28 GMT", "_created": "Sat, 28 Oct 2017 01:27:28 GMT", "_links": {"self": {"title": "User", "href": "user/59f3dd0019e7de0005a1e238"}}},
  {"username": "ltorres", "password": "123", "phone": "3435333", "_etag": "0b995ae393b45be1b6cf8025f2aeb26e30a0e9ee", "_id": "59f65975b18b1b0005660c9f", "_updated": "Sun, 29 Oct 2017 22:43:01 GMT", "_created": "Sun, 29 Oct 2017 22:43:01 GMT", "_links": {"self": {"title": "User", "href": "user/59f65975b18b1b0005660c9f"}}}]}
  ```

  **NOTA ACLARATORIA:** Debe recordarse que los archivos almacenados en la máquina virtual son temporales y al reiniciar o apagar el nodo estos se perderań. Los servicios que dependan de algún archivo no podrán iniciarse al no encontrarlo en la ruta especificada. Si por algún motivo se debio reiniciar el nodo, debe descargar nuevamente los archivos necesarios para el inicio del servicio, para el ejemplo anterior, los archivos run.py y settings.py.

---

**Despliegue del servicio de Analítica de Datos**

---

Básicamente el **Servicio de Analítica de Datos** consiten en _Jupyter Lab_ y Jupyter Notebooks\_ \(Miniconda\) con los siguiente paquetes instalados:

* bokeh
* numpy
* matplotlib
* pandas
* scikit-learn
* scikit-image
* jupyter
* jupyterlab
* ipywidgets
* numba
* pyproj
* scipy
* seaborn
* sqlite
* pymongo
* tensorflow
* zlib
* pymc3
* motionless
* utm

El servicio de **Analítica de Datos** requiere que al inicializarse se pase como variable de entorno los datos para la conexión a **MongoDB** y el nombre de la base de datos sobre la cual se realizará la analítica.

* Se recomienda crear un nuevo volumen para almacenar los notebooks que se generen en el servicio de analítica.

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker volume create --name analitica_customer1]
  ```

* Para inicializar el servicio de analítica:

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker service create --name analitica_cust1 --constraint 'node.hostname == manager' --publish 8888:8888 --env MONGO_HOST=mongo_eve --env MONGO_PORT=27017 --env MONGO_DBNAME=customer1_db --network services_overlay --mount type=volume,source=analitica_customer1,target=/home/analytics/ localhost:5000/analitica_datos]
  ```

  ** Descripción de los parámetros del servicio de Eve**

  * --name: Nombre del servicio
  * --constraint: Especifica el nombre de la máquina en la cual se desplegará el servicio
  * --publish: Puerto a mapear. \(Número de puerto del Swarm: Número de puerto del  servicio\)
  * --env: El servicio de Eve requiere de tres variables de ambiente; MONGO\_HOST: Nombre del servicio de MongoDB, MONGO\_PORT: Puerto de Mongo \(Puerto 27017 por defecto\) y MONGO\_DBNAME: Nombre de la base de datos del customer1.
  * --network: Nombre de la red a la que se enlazará el servicio. Para este caso se usa una red overlay con el fin de comunicar servicios
  * --mount: Folders o Volumenes a montar dentro del contenedor que ofrece el servicio. Para este caso es el volumen en donde se almacenaran los notebooks u otros archivos generados desde el servicio de analísis de datos.

* Se debe verificar que el servicio de **Análitica de Datos** esta ejecutado correctamente.

  ```
  **[terminal]
  **[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker service ls]
  ID                  NAME                MODE                REPLICAS            IMAGE                                   PORTS
  **[warning ywufp9gbxo0d        analitica_cust1     replicated          1/1                 localhost:5000/analitica_datos:latest   *:8888->8888/tcp]
  tab0txq47kr5        eve_customer1       replicated          1/1                 localhost:5000/eve_apache:latest        *:6001->80/tcp
  l2k5841dzoq6        mongo_eve           replicated          1/1                 localhost:5000/mongo:latest             *:27017->27017/tcp
  hw74xi7gsmbq        registry            replicated          1/1                 registry:2                              *:5000->5000/tcp
  ```

* Ingresar a la dirección [http://192.168.99.100:8888](http://192.168.99.100:8888) para verficar que el servicio es valido.

![](/assets/Analitica_test.png)

* Para verificar que las variables de entorno han sido creadas correctamente y se puede establecer conexión con la base de datos, se crea un nuevo Notebook con el siguiente contenido:

![](/assets/analitica.png)

En la gráfica se observa que se ha impreso el contenido de las variables de entorno, realizado la conexión con la base de datos y consultado la colección _user_. Esta colección es la que contiene los datos que se han insertado mediante el servidor _Eve_ y que han sido enviados usando un archivo _json_ y un archivo _json_ comprimido _gzip._

### CASOS DE USO - AVANZADO

#### CASO 1

* Un servicio de Mongo.
* Un servicio de Eve para el _customer1_ con una sola replica en el puerto 6001.
* Un servicio de Eve para el _customer2_ dos replicas en el puerto 6002.
* Un servicio de Analítica de Datos.

#### CASO 2

* Un servicio de Mongo corriendo permanentemente.
* Un servicio de Eve corriendo de forma permanente
* Un servicio de analítica corriendo de forma ocasional.

#### CASO 3

* Un servicio de Mongo corriendo permanentemente.
* Un servicio de Eve para el _customer1_ corriendo permanentemente.
* Un servicio de Eve para el _customer2_ corriendo de forma ocasional.
* Servicio de Analítica de Datos corriendo de forma ocasional.



