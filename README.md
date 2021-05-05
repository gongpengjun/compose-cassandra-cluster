# Cassandra Cluster in Docker #
A `docker-compose` blueprint that describes a 3 node Cassandra cluster.
It only exposes important Cassandra ports on the seed node to the host
machine. Internally, all of the nodes will be a part of the same Docker
network and will form a cluster using that Docker network.

## Instructions ##
In order to bring up the cluster:
- Use `docker-compose -p my up` to see the logs of all the containers
- Use `docker-compose -p my up -d` if you want it to run in the foreground

In order to check the cluster, use `docker-compose -p my ps`

In order to clean up the cluster, use `docker-compose -p my down`

指定`-p my` 则用`my`作为集群各节点的前缀，否则使用`compose-cassandra-cluster`作为前缀太长了。

## Usage Demo

### Launch Cassandra cluster

```shell
$ docker-compose -p my up -d
Creating network "my_default" with the default driver
Creating cassandra-seed-node ... done
Creating my_cassandra-node-1_1 ... done
Creating my_cassandra-node-2_1 ... done
```

### Check Cassandra cluster

```shell
$ docker-compose -p my ps
        Name                       Command               State                                             Ports
-----------------------------------------------------------------------------------------------------------------------------------------------------------
cassandra-seed-node     docker-entrypoint.sh cassa ...   Up     7000/tcp, 7001/tcp, 0.0.0.0:7199->7199/tcp, 0.0.0.0:9042->9042/tcp, 0.0.0.0:9160->9160/tcp
my_cassandra-node-1_1   docker-entrypoint.sh /bin/ ...   Up     7000/tcp, 7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp
my_cassandra-node-2_1   docker-entrypoint.sh /bin/ ...   Up     7000/tcp, 7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp
```

### Clean Cassandra cluster

```shell
$ docker-compose -p my down
Stopping my_cassandra-node-1_1 ... done
Stopping my_cassandra-node-2_1 ... done
Stopping cassandra-seed-node   ... done
Removing my_cassandra-node-1_1 ... done
Removing my_cassandra-node-2_1 ... done
Removing cassandra-seed-node   ... done
Removing network my_default
```

## Notes ##
You need to make sure that the Docker daemon has enough of resources
otherwise you will encounter exit code 137 (Out of Memory Killer) on
your containers.

When you create a single node cluster and you try to add more nodes to
the cluster, you must add them one by one. This means that you cannot
have multiple nodes join the cluster (by pointing to the seed node) at
the same time. You must add a cluster, wait for it to join the ring and
stabilize before you can begin to add another cluster. This is why you
will see an additional delay on start up between the non-seed nodes.
If you attempt to join a new node whilst stabilization has not yet been
achieved, you will see an error like this:

```
ERROR [main] 2017-08-22 23:19:11,055 CassandraDaemon.java:706 - Exception encountered during startup
java.lang.UnsupportedOperationException: Other bootstrapping/leaving/moving nodes detected, cannot bootstrap while cassandra.consistent.rangemovement is true
```

This [article](http://thelastpickle.com/blog/2017/05/23/auto-bootstrapping-part1.html)
discusses the implications of turning `cassandra.consistent.rangemovement`
off.

### Credits ###
- [The Last Pickle](http://thelastpickle.com/blog)
- [calvinlfer](https://github.com/calvinlfer)/**[compose-cassandra-cluster](https://github.com/calvinlfer/compose-cassandra-cluster)**