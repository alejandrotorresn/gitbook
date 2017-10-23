# CASOS DE USO -SWARM MODE

Para los casos de uso se implementaran un solo nodo **manager **y tres nodos **worker**. Estos nodos se crearan mediante **docker-machine** y en _VirtualBox_.

## REQUISITOS PREVIOS

1. Creación de las máquinas virtuales 

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

* ganglia/swarm management/query web interface



