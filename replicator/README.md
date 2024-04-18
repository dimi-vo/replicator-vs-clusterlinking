# Replicator

This is an example of setting up Replicator between two clusters.

- **Cluster A**: Source Cluster
- **Cluster B**: Destination Cluster

The goal is to create a mirror topic by deploying Replicator as a connector.

## Steps

### 1. Start the cluster

```shell
docker compose up -d
```

### 2. Create dummy data in the source cluster

```shell

docker exec -it broker-A-1 /bin/bash

kafka-topics --create --topic dummy --bootstrap-server broker-A-1:9092

kafka-console-producer --topic dummy --bootstrap-server broker-A-1:9092
>one
>two
>three
```

### 3. Run Replicator as a connector

Run the first request in `connect.http` to deploy Replicator as a connector.
