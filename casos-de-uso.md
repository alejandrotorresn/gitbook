# CASOS DE USO -SWARM MODE

Para los casos de uso se implementaran un solo nodo **manager **y tres nodos **worker**. Estos nodos se crearan mediante **docker-machine** y en _VirtualBox_.

Se implementaran los servicios de **MongoDB**, un servidor REST basado en **Eve **junto con una configuración personalizada de **Apache** que permite recibir archivos .gzip en el servicio REST y un contedor de **Analítica de Datos** \(Jupyter Notebook y Jupyter Lab\).

## REQUISITOS PREVIOS

1. Creación de las máquinas virtuales en _VirtualBox_
2. Creación de las imagenes necesarias. P 

## CASO 1.

* doc must include bootstrap with dockermachine

* two analytics customers

* 4 physical machines

deployment:

* 1 mongo service

* eve service for customer1 \(with only one instance==replica\) at port XXXX

* eve service for customer2 \(with two replicas\) at port YYYY

* 1 analytics service

use case2:

1. full installation

2. mongo + eves running permanently, ocasional analytics container

3. mongo + eve customer1 permanently. eve customer2 + analytics ocasional

4. ganglia/swarm management/query web interface



