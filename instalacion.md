# INSTALACIÓN DE DOCKER EN LINUX

## UBUNTU

### REQUISITOS PREVIOS

* Instalación de Ubuntu 16.04 o 17.04 de 64 bits
* Cuenta de usuario con privilegios de administracion \(sudo\)

### INSTALACIÓN DE DOCKER

Existe la posibilidad que el paquete proporcionado por los repositorios de Ubuntu no sea la última versión por lo tanto se hará la instalación desde el repositorio oficial de Docker.

A continuación se detallan los pasos de la instalación:

* Actualización de la base de datos de los repositorios de Ubuntu.

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter  $ ]**[command sudo apt-get update -y]
...
Reading package lists... Done
```

* Agregar la clave GPG del repositorio oficial de Docker:

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
]
Executing: /tmp/apt-key-gpghome.QbF5xSF57C/gpg.1.sh --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
gpg: key F76221572C52609D: "Docker Release Tool (releasedocker) <docker@docker.com>" 1 new signature
gpg: Total number processed: 1
gpg:         new signatures: 1
```

* Agregar el repositorio oficial de Docker a las repositorios de Ubuntu:

 Para Ubuntu 16.04

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command sudo apt-add-repository 'deb https://apt.dockerproject.org/repo ubuntu-xenial main']
```

 Para Ubuntu 17.04

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command sudo apt-add-repository 'deb https://apt.dockerproject.org/repo ubuntu-zesty main']
```

* Actualizar los repositorios de Ubuntu:

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command sudo apt-get update -y]
```

* Verificar si el paquete de instalación de Docker proviene de los repositorios oficiales:

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command apt-cache policy docker-engine]
docker-engine:
  Installed: (none)
  Candidate: 17.05.0~ce-0~ubuntu-zesty
  Version table:
 *** 17.05.0~ce-0~ubuntu-zesty 500
        500 https://apt.dockerproject.org/repo ubuntu-zesty/main amd64 Packages
        100 /var/lib/dpkg/status
```

* Instalar Docker Engine:

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command sudo apt-get install -y docker-engine]
```

* Verificar que docker engine se este ejecutando:

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command sudo systemctl status docker]
    docker.service - Docker Application Container Engine
       Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
       Active: active (running) since Sun 2017-10-22 12:55:02 -05; 5h 47min ago
       Docs: https://docs.docker.com
       Main PID: 1802 (dockerd)
```

* Se recomienda configurar el comando **docker** para ser usado sin la necesidad del somando sudo:

 ```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command sudo usermod -aG docker $(whoami)]
```

 Se debe reiniciar la sesión para que este cambio sea efectivo.

