---

title: Redis Cluster 구축 (Mac, Rocky Linux)
date: 2024-08-29
categories: [RedisCluster]
tags: [RedisCluster]
layout: post
toc: true
math: true
mermaid: true

---

## 공식문서

[레디스 공식문서 - 클러스터링](https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/)

## Docker Compose [MAC]

```yaml
services:
  redis-master-1:
    container_name: redis-master-1
    image: redis:7.2
    command: ["redis-server", "--port", "6379", "--cluster-enabled", "yes", "--cluster-config-file", "nodes.conf", "--cluster-node-timeout", "5000", "--appendonly", "yes"]
    restart: always
    ports:
      - 6379:6379
      - 6479:6479
      - 6579:6579
      - 6380:6380
      - 6480:6480
      - 6580:6580

  redis-master-2:
    network_mode: "service:redis-master-1"
    container_name: redis-master-2
    image: redis:latest
    command: ["redis-server", "--port", "6479", "--cluster-enabled", "yes", "--cluster-config-file", "nodes.conf", "--cluster-node-timeout", "5000", "--appendonly", "yes"]
    restart: always

  redis-master-3:
    network_mode: "service:redis-master-1"
    container_name: redis-master-3
    image: redis:latest
    command: ["redis-server", "--port", "6579", "--cluster-enabled", "yes", "--cluster-config-file", "nodes.conf", "--cluster-node-timeout", "5000", "--appendonly", "yes"]
    restart: always

  redis-slave-1:
    network_mode: "service:redis-master-1"
    container_name: redis-slave-1
    image: redis:latest
    command: ["redis-server", "--port", "6380", "--cluster-enabled", "yes", "--cluster-config-file", "nodes.conf", "--cluster-node-timeout", "5000", "--appendonly", "yes"]
    restart: always

  redis-slave-2:
    network_mode: "service:redis-master-1"
    container_name: redis-slave-2
    image: redis:latest
    command: ["redis-server", "--port", "6480", "--cluster-enabled", "yes", "--cluster-config-file", "nodes.conf", "--cluster-node-timeout", "5000", "--appendonly", "yes"]
    restart: always

  redis-slave-3:
    network_mode: "service:redis-master-1"
    container_name: redis-slave-3
    image: redis:latest
    command: ["redis-server", "--port", "6580", "--cluster-enabled", "yes", "--cluster-config-file", "nodes.conf", "--cluster-node-timeout", "5000", "--appendonly", "yes"]
    restart: always

  redis_cluster_entry:
    network_mode: "service:redis-master-1"
    image: redis:latest
    container_name: redis_cluster_entry
    command: redis-cli --cluster create 127.0.0.1:6379 127.0.0.1:6479 127.0.0.1:6579 127.0.0.1:6380 127.0.0.1:6480 127.0.0.1:6580 --cluster-yes
    depends_on:
      - redis-master-1
      - redis-master-2
      - redis-master-3
      - redis-slave-1
      - redis-slave-2
      - redis-slave-3
```

## Docker Compose [Linux]

```yaml
services:
    redis-master-1:
        network_mode: "host"
        container_name: redis-master-1
        image: redis:latest
        command: redis-server --port 6379 --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes
        volumes:
            - redis-master-1-data:/data

    redis-master-2:
        network_mode: "host"
        container_name: redis-master-2
        image: redis:latest
        command: redis-server --port 6479 --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes
        volumes:
            - redis-master-2-data:/data

    redis-master-3:
        network_mode: "host"
        container_name: redis-master-3
        image: redis:latest
        command: redis-server --port 6579 --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes
        volumes:
            - redis-master-3-data:/data

    redis-slave-1:
        network_mode: "host"
        container_name: redis-slave-1
        image: redis:latest
        command: redis-server --port 6380 --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes
        volumes:
            - redis-slave-1-data:/data

    redis-slave-2:
        network_mode: "host"
        container_name: redis-slave-2
        image: redis:latest
        command: redis-server --port 6480 --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes
        volumes:
            - redis-slave-2-data:/data

    redis-slave-3:
        network_mode: "host"
        container_name: redis-slave-3
        image: redis:latest
        command: redis-server --port 6580 --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes
        volumes:
            - redis-slave-3-data:/data

    redis_cluster_entry:
        network_mode: "host"
        image: redis:latest
        container_name: redis_cluster_entry
        command: >
            bash -c "
              sleep 10 &&
              redis-cli --cluster create {serverIPAddress}:6379 {serverIPAddress}:6479 {serverIPAddress}:6579 {serverIPAddress}:6380 {serverIPAddress}:6480 {serverIPAddress}:6580 --cluster-replicas 1 --cluster-yes
            "
        depends_on:
            - redis-master-1
            - redis-master-2
            - redis-master-3
            - redis-slave-1
            - redis-slave-2
            - redis-slave-3

volumes:
    redis-master-1-data:
    redis-master-2-data:
    redis-master-3-data:
    redis-slave-1-data:
    redis-slave-2-data:
    redis-slave-3-data:
```

---

## Spring Boot에서 클러스터 접속

### Config

```java
@Configuration
@RequiredArgsConstructor
public class RedisConfig {

    private final RedisInfo redisInfo;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(new RedisClusterConfiguration(redisInfo.getNodes()));
    }
}

@Data
@Configuration
@ConfigurationProperties(prefix = "spring.data.redis.cluster")
public class RedisInfo {
    private List<String> nodes;
}
```

### yml

```yaml
spring:
  data:
    redis:
      cluster:
        nodes:
          - localhost:6379
          - localhost:6479
          - localhost:6579
          - localhost:6380
          - localhost:6480
          - localhost:6580
```
