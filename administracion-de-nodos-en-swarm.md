# ADMINISTRACIÓN DE LOS NODOS EN SWARM

Ingresar al nodo manager \(**manager1**\):

```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine ssh manager1]
```

## Listar los nodos de Swarm

```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker node ls]
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
e8f5g2m21pindc15dge3qb4ja *   manager1            Ready               Active              Leader
wio0db8ezfaw8e1yyi5pm1k1t     worker1             Ready               Active              
```

**NOTA:** El asterisco \(\*\) indica el nodo activo. Para este caso el _**manager1**_.

## Inspecionar un nodo

```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker node inspect worker1 --pretty]
ID:                     wio0db8ezfaw8e1yyi5pm1k1t
Hostname:               worker1
Joined at:              2017-10-25 15:36:07.71818709 +0000 utc
Status:
 State:                 Ready
 Availability:          Active
 Address:               192.168.99.101
Platform:
 Operating System:      linux
 Architecture:          x86_64
Resources:
 CPUs:                  1
 Memory:                995.8MiB
Plugins:
 Log:           awslogs, fluentd, gcplogs, gelf, journald, json-file, logentries, splunk, syslog
 Network:               bridge, host, macvlan, null, overlay
 Volume:                local
Engine Version:         17.10.0-ce
Engine Labels:
 - provider=virtualbox
```


## Obtener los comandos para agregar nuevos workers o managers

**Workers**

```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker swarm join-token worker]
To add a worker to this swarm, run the following command:

    **[warning docker swarm join --token SWMTKN-1-16b0v848cdwo4ehoxcxwmxq3fe2havmm7xacs2u4ilxjrkkfvn-3rq4jjhizy9sw6ive1vhgyqxl 192.168.99.100:2377]
```

**Managers**

```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker swarm join-token manager]
To add a manager to this swarm, run the following command:

    **[warning docker swarm join --token SWMTKN-1-16b0v848cdwo4ehoxcxwmxq3fe2havmm7xacs2u4ilxjrkkfvn-d5taz1ij30ch7mzan8t9g57i3 192.168.99.100:2377]
```

## Obener solo el token

```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker swarm join-token --quiet worker]
**[warning SWMTKN-1-16b0v848cdwo4ehoxcxwmxq3fe2havmm7xacs2u4ilxjrkkfvn-3rq4jjhizy9sw6ive1vhgyqxl]
```

## Regenerar los tokens

```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker swarm join-token --rotate worker]
Successfully rotated worker join token.

To add a worker to this swarm, run the following command:

    **[warning docker swarm join --token SWMTKN-1-16b0v848cdwo4ehoxcxwmxq3fe2havmm7xacs2u4ilxjrkkfvn-c9cfisa9snnirw2dz0vwy7ul2 192.168.99.100:2377]
```
