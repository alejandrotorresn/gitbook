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

![](/assets/1.png)

![](/assets/2.png)

![](/assets/3.png)

![](/assets/4.png)

![](/assets/5.png)

![](/assets/6.png)

![](/assets/7.png)

![](/assets/8.png)

![](/assets/9.png)

![](/assets/10.png)

![](/assets/11.png)

```
docker@manager1:~$ docker service create --name Sync_worker1 --constraint 'node.hostname == worker1' -p 8889:8888 -p 55555 --mount type=volume,source=data_analytics,destination=/mnt/sync --restart-condition on-failure resilio/sync
```

![](/assets/12.png)

![](/assets/13.png)

![](/assets/14.png)

![](/assets/15.png)

![](/assets/16.png)

![](/assets/17.png)

![](/assets/18.png)





```
antares@pleyades:~$ docker-machine ssh manager1 sudo chown -R 999:999 /mnt/sda1/var/lib/docker/volumes/data_analytics/_data/folders
antares@pleyades:~$ docker-machine ssh worker1 sudo chown -R 999:999 /mnt/sda1/var/lib/docker/volumes/data_analytics/_data/folders

```



```
docker@manager1:$ docker service create --name analitica --constraint 'node.hostname == manager1' --publish 80:8888 --env MONGO_HOST=mongo_eve --env MONGO_PORT=27017 --env MONGO_DBNAME=customer1_db --network services_overlay --mount type=volume,source=data_analytics,target=/home/analytics/Notebook localhost:5000/analitica_datos
```



