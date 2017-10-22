# INSTALACIÓN DE DOCKER EN UBUNTU

## REQUISITOS PREVIOS

* Instalación de Ubuntu 16.04 de 64 bits
* Cuenta de usuario con privilegios de administracion \(sudo\)

## INSTALACIÓN DE DOCKER

Existe la posibilidad que el paquete proporcionado por los repositorios de Ubuntu no sea la última versión por lo tanto se hará la instalación desde el repositorio oficial de Docker.

A continuación se detallan los pasos de la instalación:

1. Actualización de la base de datos de los repositorios de Ubuntu.

```
sudo apt-get update -y
```

1. Agregar la clave GPG del repositorio oficial de Docker:

```
sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
```

1. Agregar el repositorio oficial de Docker a las repositorios de Ubuntu:

Para Ubuntu 16.04

```
sudo apt-add-repository 'deb https://apt.dockerproject.org/repo ubuntu-xenial main'
```

Para Ubuntu 17.04

```
sudo apt-add-repository 'deb https://apt.dockerproject.org/repo ubuntu-zesty main'
```

1. Actualizar los repositorios de Ubuntu:

```
sudo apt-get update -y
```

1. Verificar si el paquete de instalación de Docker proviene de los repositorios oficiales:

```
apt-cache policy docker-engine

docker-engine:
  Installed: (none)
  Candidate: 17.05.0~ce-0~ubuntu-zesty
  Version table:
 *** 17.05.0~ce-0~ubuntu-zesty 500
        500 https://apt.dockerproject.org/repo ubuntu-zesty/main amd64 Packages
        100 /var/lib/dpkg/status
```

1. Instalar Docker Engine

```
sudo apt-get install -y docker-engine
```



