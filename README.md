## Redis Cluster

#### Redis cluster with Docker Compose

Using Docker Compose to setup a redis cluster with sentinel.

## Prerequisite

Install [Docker](https://www.docker.com/) and [Docker Compose](https://docs.docker.com/compose/) in testing environment

## Docker Compose template of Redis cluster

The template defines the topology of the Redis cluster

```
services:
  redis01:
    build: redis-config
    ports:
      - '6380:6379'
    networks:
      - host
    env_file:
      - 'redis-config/env/master_variables.env'

  redis02:
    build: redis-config
    ports:
      - '6381:6379'
    depends_on:
      - redis01
    networks:
      - host
    env_file:
      - 'redis-config/env/slave_variables.env'

  redis03:
    build: redis-config
    ports:
      - '6382:6379'
    depends_on:
      - redis01
    networks:
      - host
    env_file:
      - 'redis-config/env/slave_variables.env'

  proxy:
    build: ./haproxy
    depends_on:
      - redis01
      - redis02
      - redis03
    networks:
      - host
    ports:
      - '9911:8080'
      - '8080:80'

networks:
  host:
```

## There are following services in the cluster;

**master**: Redis master

**slave**: Redis slave

The details could be found in sentinel/sentinel.conf

The default values of the environment variables for Sentinel are as following

* SENTINEL_QUORUM: 2
* SENTINEL_DOWN_AFTER: 1000
* SENTINEL_FAILOVER: 5000

## **Play with it**

Build the sentinel Docker image

`docker-compose build`

**Start the redis cluster**

`docker-compose up -d`

**Check the status of redis cluster**

`docker-compose ps`

Name                    Command               State                                       Ports
-----------------------------------------------------------------------------------------------

redis_proxy_1     /docker-entrypoint.sh hapr ...   Up      0.0.0.0:8080->80/tcp,:::8080->80/tcp, 0.0.0.0:9911->8080/tcp,:::9911->8080/tcp
redis_redis01_1   /etc/redis/redis-entrypoint.sh   Up      0.0.0.0:6380->6379/tcp,:::6380->6379/tcp
redis_redis02_1   /etc/redis/redis-entrypoint.sh   Up      0.0.0.0:6381->6379/tcp,:::6381->6379/tcp
redis_redis03_1   /etc/redis/redis-entrypoint.sh   Up      0.0.0.0:6382->6379/tcp,:::6382->6379/tcp
