# SINCRONIZAR VOLUMENES EN SWARM



```
**[terminal]
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine ssh manager1 docker volume create data_analytics]
data_analytics
**[prompt user@server]**[path ~]**[delimiter $ ]**[command docker-machine ssh worker1 docker volume create data_analytics]
data_analytics
```



```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker service create --name sync_manager1 --constraint 'node.hostname == manager1' -p 8888:8888 -p 55555 --mount type=volume,source=data_analytics,destination=/mnt/sync --restart-condition on-failure resilio/sync]
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
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker service create --name sync_worker1 --constraint 'node.hostname == worker1' -p 8889:8888 -p 55555 --mount type=volume,source=data_analytics,destination=/mnt/sync --restart-condition on-failure resilio/sync]
```


![](/assets/12.png)

![](/assets/13.png)

![](/assets/14.png)

![](/assets/15.png)

![](/assets/16.png)

![](/assets/17.png)

![](/assets/18.png)

![](/assets/19.png)

![](/assets/20.png)


```
**[terminal]
**[prompt docker@manager1]**[path ~]**[delimiter $ ]**[command docker service create --name analitica --constraint 'node.hostname == manager1' --publish 80:8888 --mount type=volume,source=data_analytics,target=/home/analytics/Notebook localhost:5000/analitica_datos]
```

