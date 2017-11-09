# SINCRONIZAR VOLUMENES EN SWARM

Para poder sincronizar los volumenes se usa en este caso la herramienta **Resilio Sync**. Se basa en la tecnología _peer to peer_ para la rápida sincronización del contenido de los dispositivos vinculados (_host_).

Para sincronizar los volumenes se realiza el siguiente procedimiento:

* Crear los volumenes en cada uno de los _host_ del **swarm**

  ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine ssh manager1 docker volume create data_analytics]
data_analytics
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine ssh worker1 docker volume create data_analytics]
data_analytics
```

- Ingresar al nodo **manager1**

  ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine ssh manager1]
```

- Lazar el servicio de **Resilio Sync** en el **manager1** montando en él, el volumen a sincronizar.

 ```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker service create --name sync_manager1 --constraint 'node.hostname == manager1' -p 8888:8888 -p 55555 --mount type=volume,source=data_analytics,destination=/mnt/sync --restart-condition on-failure resilio/sync]
```

 El servicio se ha lanzado en el puerto **8888** del host que mapea el puerto **8888** del servicio de Resilio.
 
 **Nota:** El servicio puede mapearse a cualquier puerto disponible.

- Ingresar a una de las IPs del swarm especificando el puerto asignado (http://192.168.99.100:8888)

 ![](/assets/1.png)
 
- Asignar un usuario y contraseña para el servicio de Resilio Sync.

 ![](/assets/2.png)
 
- Este contenedor contiene tanto la versión _home_ como las versiones _Business_. Para este caso se selecciona la versión **Sync Home**.
 
 ![](/assets/3.png)
 
- Asignar un nombre.

 ![](/assets/4.png)

- Una vez realizada la configuración inicial del servicio, ir al parte superior izquierda (botón +) y seleccionar **Standard folder**

 ![](/assets/5.png)
 
 ![](/assets/6.png)
 
- Cuando se crea el servicio, dentro del volumen se crean dos carpetas y un archivo, pero solo se podrá ver la carpeta **folders** que es donde se creará el contenido a sincronizar. Seleccione la carpeta **folders** y haga clic en el botón **Open**.

 ![](/assets/7.png)
 
- Seleccionar **Read & Write**

 ![](/assets/8.png)
 
- Hacer clic en **Key** y copiar la llave de **Read & Write**. Esta llave se agregará en los servicios de **Resilio Sync** corriendo en los otros host.

 ![](/assets/9.png)
 
 ![](/assets/10.png)
 
 ![](/assets/11.png)

- Lanzar el servicio de **Resilio Sync** en un nodo **worker1**. Cabe recordar que el volumen que se monta en este servicio debe existir en el host **worker1**.

  ```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker service create --name sync_worker1 --constraint 'node.hostname == worker1' -p 8889:8888 -p 55555 --mount type=volume,source=data_analytics,destination=/mnt/sync --restart-condition on-failure resilio/sync]
```

- Ingresar al servicio con alguna de las IPs del **swarm** especificando el puerto asignado (http://192.168.99.101:8889). Crear el usuario y la contraseña del servicio.

 ![](/assets/12.png)
 
- Seleccionar **Sync Home**.

 ![](/assets/13.png)
 
- Asignar un nombre.

 ![](/assets/14.png)
 
- Hacer clic en el botón de la parte superiro izquierda (+) y seleccionar **Enter a key or link** y pegar el _key_ que se obtuvo anteriormente.

 ![](/assets/15.png)
 
 ![](/assets/16.png)
 
- Seleccionar la carpeta **folders** y hacer clic en **Open**.

 ![](/assets/17.png)
 
 ![](/assets/18.png)
 
- En la parte derecha de la carpeta compartida hacer clic en **Share** y seleccionar **Read & Write**.

 ![](/assets/19.png)
 
 ![](/assets/20.png)
 
 **Nota:** Este procedimiento debe realizarse sobre cada uno de los nodos workers.
 
- Lanzar el servicio que se desee sobre cualquier nodo del swarm especificando el volumen que el servicio **Resilio Sync** esta encargado de sincronizar. 

  ```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker service create --name analitica --constraint 'node.hostname == manager1' --publish 80:8888 --mount type=volume,source=data_analytics,target=/home/analytics/Notebook localhost:5000/analitica_datos]
```



