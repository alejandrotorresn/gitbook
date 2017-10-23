# INSTALACIÓN DE DOCKER MACHINE

* \[Instalar Docker \(docker engine\)\]\(../chapter1/instalacion.md\)

* Descargar el binario de **Docker Machine** y habiliatarlo en el PATH

  $ curl -L [https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-\`uname](https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-`uname) -s-uname -m\` &gt;/tmp/docker-machine && chmod +x /tmp/docker-machine && sudo cp /tmp/docker-machine /usr/local/bin/docker-machine

* Verificar la Instalación:

```
$ docker-machine version

    docker-machine version 0.12.2, build 9371605
```



