# Red Overlay sin Swarm

Una de las cosas que se debe tener en cuenta al trabajar con multiples host en Docker es que cada uno de ellos no pueden verse entre sí. Por lo cual, no se le es permitido a los host compartir ningún recurso, ni siquiera una red. Pero existe una forma de hacerlo y es crear un servicio especial que tenga como única tarea mantener una lista de host involucrados, la configuración de la red y que le diga a los _Do_ocker Engine_ _como deben usarla.

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

Docker permite dos formas de conectarse al _host_ creado. La primera de ellas es usar _**docker-machine ssh &lt;nombre\_maquina&gt; **_con lo cual se accede de forma directa al host. La segunda forma, permite conectar el  Docker Engine local con el del host remoto, en otras palabras, los comandos ejecutados localmente se ejecutaran sobre el host.

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

Desde un navegador puede entrarse a la dirección ip resultante de la ejecuación del comando anterior junto con el puerto 8500 \(**http://192.168.99.104:8500**\)

### 











