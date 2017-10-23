# ADMINISTRACIÓN DE LOS NODOS EN SWARM

Ingresar al nodo manager \(**manager1**\):

```
$ docker-machine ssh manager1
```

* **Listar los nodos de Swarm**

```
$ docker node ls

ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
i7l5245yf58tinn2hsab39d6i *   manager1            Ready               Active              Leader
nj1px46tmntlasu7ykhlruv7j     worker1             Ready               Active
```

**NOTA:** El asterisco \(\*\) indica el nodo activo. Para este caso el _**manager1**_.

* **Inspecionar un nodo**

```
$ docker node inspect worker1 --pretty


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

### View the join command or update a swarm join token

```
docker@manager1:~$ docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-61hpva2ixi24x1dzkfs61y7a5nwuuoq8c2n6onfdowg2knaphv-e8p88sf6r4x0407ek959rvgfj 192.168.99.100:2377

docker@manager1:~$ docker swarm join-token manager
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-61hpva2ixi24x1dzkfs61y7a5nwuuoq8c2n6onfdowg2knaphv-brtem0czo5f7d46s7uponsfoy 192.168.99.100:2377
```

#  {#title}

Only token

```
docker@manager1:~$ docker swarm join-token --quiet worker
SWMTKN-1-61hpva2ixi24x1dzkfs61y7a5nwuuoq8c2n6onfdowg2knaphv-e8p88sf6r4x0407ek959rvgfj
```

to invalidate the old token and generate a new token

```
$docker swarm join-token  --rotate worker

To add a worker to this swarm, run the following command:

    docker swarm join \
--token SWMTKN-1-2kscvs0zuymrsc9t0ocyy1rdns9dhaodvpl639j2bqx55uptag-ebmn5u927reawo27s3azntd44 \
    192.168.99.100:2377
```



