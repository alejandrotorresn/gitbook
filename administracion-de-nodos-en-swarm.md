# ADMINISTRACIÓN DE LOS NODOS EN SWARM

* Ingresar al nodo manager \(**manager1**\):

```
$ docker-machine ssh manager1
```

**Listar los nodos de Swarm**

```
$ docker node ls

ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
i7l5245yf58tinn2hsab39d6i *   manager1            Ready               Active              Leader
nj1px46tmntlasu7ykhlruv7j     worker1             Ready               Active
```

**NOTA:** El asterisco \(\*\) indica el nodo activo. Para este caso el _**manager1**_.

## Inspect an individual node {#inspect-an-individual-node}

```
docker@manager1:~$ docker node inspect worker1 --pretty

ID:                     nj1px46tmntlasu7ykhlruv7j
Hostname:               worker1
Joined at:              2017-10-19 14:43:38.495408067 +0000 utc
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



