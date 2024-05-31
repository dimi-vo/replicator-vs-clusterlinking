# Cluster Linking

This is an example of setting up Cluster Linking between two clusters.

- **Cluster A**: Source Cluster
- **Cluster B**: Destination Cluster

The `compose.yaml` file will spin up two clusters with one broker and one controller in each, as well as one Confluent Control Center instance.

The config folder contains the configuration file for the cluster link.

No encryption is used in this example.

## Steps

### 1. Start the containers

```shell
docker compose up -d
```

### 2. Enable CL in destination cluster

```shell
docker exec -it broker-B-1 /bin/bash

echo "confluent.cluster.link.metadata.topic.replication.factor=1" >> /etc/kafka/kraft/broker.properties

sed -i '' -e "s/confluent.cluster.link.enable=false/confluent.cluster.link.enable=true/g" /etc/kafka/kraft/broker.properties
```

### 3. Restart the broker

```shell
/bin/krafka-server-stop

docker compose up -d
```

### 4. Create dummy data in the source cluster

```shell

docker exec -it broker-A-1 /bin/bash

kafka-topics --create --topic dummy --bootstrap-server broker-A-1:9092

kafka-console-producer --topic dummy --bootstrap-server broker-A-1:9092
>one
>two
>three
```

### 5. Create the link and start the mirroring process in the destination cluster

In a new terminal session we start the link from the destination side.

```shell
docker exec -it broker-B-1 /bin/bash

kafka-cluster-links --bootstrap-server broker-B-1:9092 --create --link demo-link --config-file link.config

kafka-mirrors --create --mirror-topic dummy --link demo-link --bootstrap-server broker-B-1:9092 --config retention.ms=600000

# Consume the mirror topic
kafka-console-consumer --topic demo --from-beginning --bootstrap-server broker-B-1:9093
```

### 6. Cleanup

```shell
kafka-topics --bootstrap-server broker-B-1:9092 --delete --topic dummy

kafka-cluster-links --bootstrap-server broker-B-1:9092 --delete --link demo-link
```
