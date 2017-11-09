# SINCRONIZAR VOLUMENES EN SWARM





```
antares@pleyades:~$ docker-machine ssh manager1 docker volume create data_analytics
data_analytics
antares@pleyades:~$ docker-machine ssh worker1 docker volume create data_analytics
data_analytics
antares@pleyades:~$ 
```





```
docker@manager1:~$ docker service create --name Sync_manager1 --constraint 'node.hostname == manager1' -p 8888:8888 -p 55555 --mount type=volume,source=data_analytics,destination=/mnt/sync --restart-condition on-failure resilio/sync
```



```
docker@manager1:~$ docker service create --name Sync_worker1 --constraint 'node.hostname == worker1' -p 8889:8888 -p 55555 --mount type=volume,source=data_analytics,destination=/mnt/sync --restart-condition on-failure resilio/sync
```







