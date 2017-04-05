# <a name="about"></a>About

This image contains an installation of Cassandra3, open-source distributed storage system.
[Official Image Launcher Page](https://console.cloud.google.com/launcher/details/google/cassandra3).

Pull command:
```shell
gcloud docker -- pull gcr.io/orbitera-dev/cassandra3
```

Dockerfile for this image can be found [here](https://github.com/GoogleCloudPlatform/cassandra-docker/tree/master/3.10/).

# <a name="table-of-contents"></a>Table of Contents
* [Using Kubernetes](#using-kubernetes)
  * [How to use cassandra server instance](#how-to-use-cassandra-server-instance-kubernetes)
    * [Start single Cassandra container](#start-single-cassandra-container-kubernetes)
    * [Connect to cassandra from cqlsh](#connect-to-cassandra-from-cqlsh-kubernetes)
    * [Make a Cassandra cluster](#make-a-cassandra-cluster-kubernetes)
    * [Run with persistent data volumes](#run-with-persistent-data-volumes-kubernetes)
* [Using Docker](#using-docker)
  * [How to use cassandra server instance](#how-to-use-cassandra-server-instance-docker)
    * [Start single Cassandra container](#start-single-cassandra-container-docker)
    * [Connect to cassandra from cqlsh](#connect-to-cassandra-from-cqlsh-docker)
    * [Make a Cassandra cluster](#make-a-cassandra-cluster-docker)
    * [Run with persistent data volumes](#run-with-persistent-data-volumes-docker)
* [References](#references)
  * [Ports](#references-ports)
  * [Environment Variables](#references-environment-variables)
  * [Volumes](#references-volumes)

# <a name="using-kubernetes"></a>Using Kubernetes

## <a name="how-to-use-cassandra-server-instance-kubernetes"></a>How to use cassandra server instance

This section describes how to spin up a Cassandra service using this image.

### <a name="start-single-cassandra-container-kubernetes"></a>Start single Cassandra container

Copy the following content to `pod.yaml` file, and run `kubectl create -f pod.yaml`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: some-cassandra
  labels:
    name: some-cassandra
spec:
  containers:
    - image: cassandra
      name: cassandra
```

Run the following to expose the ports:
```shell
kubectl expose pod some-cassandra --name some-cassandra-7000 \
  --type LoadBalancer --port 7000 --protocol TCP
kubectl expose pod some-cassandra --name some-cassandra-7001 \
  --type LoadBalancer --port 7001 --protocol TCP
kubectl expose pod some-cassandra --name some-cassandra-7199 \
  --type LoadBalancer --port 7199 --protocol TCP
kubectl expose pod some-cassandra --name some-cassandra-9042 \
  --type LoadBalancer --port 9042 --protocol TCP
kubectl expose pod some-cassandra --name some-cassandra-9160 \
  --type LoadBalancer --port 9160 --protocol TCP
```

Cassandra will be accessible on your localhost using `cqlsh`

For information about how to retain your Cassandra data across restarts, see [Run with persistent data volumes](#run-with-persistent-data-volumes-kubernetes).

### <a name="connect-to-cassandra-from-cqlsh-kubernetes"></a>Connect to cassandra from cqlsh

The following command starts another Cassandra container instance and runs cqlsh (Cassandra Query Language Shell) against your original Cassandra container, allowing you to execute CQL statements against your database instance.

```shell
kubectl run \
  some-cqlsh \
  --image cassandra \
  --rm --attach --restart=Never \
  -it \
  -- sh -c "exec cqlsh $(kubectl get pods -o wide | awk '/^some-cassandra/ {print $6}')"
```

### <a name="make-a-cassandra-cluster-kubernetes"></a>Make a Cassandra cluster

The following command starts another Cassandra container instance and runs cqlsh (Cassandra Query Language Shell) against your original Cassandra container, allowing you to execute CQL statements against your database instance.

Copy the following content to `pod.yaml` file, and run `kubectl create -f pod.yaml`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: some-cluster-n1
  labels:
    name: some-cluster-n1
spec:
  containers:
    - image: cassandra
      name: cluster-n1
      env:
        - name: CASSANDRA_SEEDS
          value: some-cassandra-1-ip some-cassandra-2-ip
```

### <a name="run-with-persistent-data-volumes-kubernetes"></a>Run with persistent data volumes

We can store data on persistent volumes Cassandra This way the installation remains intact across restarts.

Copy the following content to `pod.yaml` file, and run `kubectl create -f pod.yaml`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: some-cassandra
  labels:
    name: some-cassandra
spec:
  containers:
    - image: cassandra
      name: cassandra
      volumeMounts:
        - name: cassandra-data
          mountPath: /var/lib/cassandra
  volumes:
    - name: cassandra-data
      persistentVolumeClaim:
        claimName: cassandra-data
---
# Request a persistent volume from the cluster using a Persistent Volume Claim.
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: cassandra-data
  annotations:
    volume.alpha.kubernetes.io/storage-class: default
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 5Gi
```

Run the following to expose the ports:
```shell
kubectl expose pod some-cassandra --name some-cassandra-7000 \
  --type LoadBalancer --port 7000 --protocol TCP
kubectl expose pod some-cassandra --name some-cassandra-7001 \
  --type LoadBalancer --port 7001 --protocol TCP
kubectl expose pod some-cassandra --name some-cassandra-7199 \
  --type LoadBalancer --port 7199 --protocol TCP
kubectl expose pod some-cassandra --name some-cassandra-9042 \
  --type LoadBalancer --port 9042 --protocol TCP
kubectl expose pod some-cassandra --name some-cassandra-9160 \
  --type LoadBalancer --port 9160 --protocol TCP
```

# <a name="using-docker"></a>Using Docker

## <a name="how-to-use-cassandra-server-instance-docker"></a>How to use cassandra server instance

This section describes how to spin up a Cassandra service using this image.

### <a name="start-single-cassandra-container-docker"></a>Start single Cassandra container

Use the following content for the `docker-compose.yml` file, then run `docker-compose up`.
```yaml
version: '2'
services:
  cassandra:
    image: cassandra
    ports:
      - '7000:7000'
      - '7001:7001'
      - '7199:7199'
      - '9042:9042'
      - '9160:9160'
```

Or you can use `docker run` directly:

```shell
docker run \
  --name some-cassandra \
  -p 7000:7000 \
  -p 7001:7001 \
  -p 7199:7199 \
  -p 9042:9042 \
  -p 9160:9160 \
  -d \
  cassandra
```

Cassandra will be accessible on your localhost using `cqlsh`

For information about how to retain your Cassandra data across restarts, see [Run with persistent data volumes](#run-with-persistent-data-volumes-docker).

### <a name="connect-to-cassandra-from-cqlsh-docker"></a>Connect to cassandra from cqlsh

The following command starts another Cassandra container instance and runs cqlsh (Cassandra Query Language Shell) against your original Cassandra container, allowing you to execute CQL statements against your database instance.

### <a name="make-a-cassandra-cluster-docker"></a>Make a Cassandra cluster

The following command starts another Cassandra container instance and runs cqlsh (Cassandra Query Language Shell) against your original Cassandra container, allowing you to execute CQL statements against your database instance.

Use the following content for the `docker-compose.yml` file, then run `docker-compose up`.
```yaml
version: '2'
services:
  cluster-n1:
    image: cassandra
    environment:
      CASSANDRA_SEEDS: some-cassandra-1 some-cassandra-2...
```

Or you can use `docker run` directly:

```shell
docker run \
  --name some-cluster-n1 \
  -e CASSANDRA_SEEDS=$(docker inspect --format='{{ .NetworkSettings.IPAddress }}' some-cassandra) \
  -d \
  cassandra
```

### <a name="run-with-persistent-data-volumes-docker"></a>Run with persistent data volumes

We can store data on persistent volumes, whis way the installation remains intact across restarts. Assume that `/var/lib/docker/volumes` is the persistent directory on the host.

Use the following content for the `docker-compose.yml` file, then run `docker-compose up`.
```yaml
version: '2'
services:
  cassandra:
    image: cassandra
    ports:
      - '7000:7000'
      - '7001:7001'
      - '7199:7199'
      - '9042:9042'
      - '9160:9160'
    volumes:
      - /var/lib/docker/volumes:/var/lib/cassandra
```

Or you can use `docker run` directly:

```shell
docker run \
  --name some-cassandra \
  -p 7000:7000 \
  -p 7001:7001 \
  -p 7199:7199 \
  -p 9042:9042 \
  -p 9160:9160 \
  -v /var/lib/docker/volumes:/var/lib/cassandra \
  -d \
  cassandra
```

# <a name="references"></a>References

## <a name="references-ports"></a>Ports

These are the ports exposed by the container image.

| **Port** | **Description** |
|:---------|:----------------|
| TCP 7000 | Standard port for cluster communication. |
| TCP 7001 | SSL port for cluster communication. |
| TCP 7199 | JMX clients. |
| TCP 9042 | Native protocol clients. |
| TCP 9160 | Deprecated Thrift interface. |

## <a name="references-environment-variables"></a>Environment Variables

These are the environment variables understood by the container image.

| **Variable** | **Description** |
|:-------------|:----------------|
| CASSANDRA_LISTEN_ADDRESS | This variable is for controlling which IP address to listen for incoming connections on. The default value is "auto", which will set the listen_address option in cassandra.yaml to the IP address of the container as it starts. This default should work in most use cases. |
| CASSANDRA_BROADCAST_ADDRESS | This variable is for controlling which IP address to advertise to other nodes. The default value is the value of `CASSANDRA_LISTEN_ADDRESS`. It will set the broadcast_address and broadcast_rpc_address options in cassandra.yaml. |
| CASSANDRA_RPC_ADDRESS | This variable is for controlling which address to bind the thrift rpc server to. If you do not specify an address, the wildcard address (0.0.0.0) will be used. It will set the rpc_address option in cassandra.yaml. |
| CASSANDRA_START_RPC | This variable is for controlling if the thrift rpc server is started. It will set the start_rpc option in cassandra.yaml. |
| CASSANDRA_SEEDS | This variable is the comma-separated list of IP addresses used by gossip for bootstrapping new nodes joining a cluster. It will set the seeds value of the seed_provider option in cassandra.yaml. The `CASSANDRA_BROADCAST_ADDRESS` will be added the the seeds passed in so that the sever will talk to itself as well. |
| CASSANDRA_CLUSTER_NAME | This variable sets the name of the cluster and must be the same for all nodes in the cluster. It will set the cluster_name option of cassandra.yaml. |
| CASSANDRA_NUM_TOKENS | This variable sets number of tokens for this node. It will set the num_tokens option of cassandra.yaml. |
| CASSANDRA_DC | This variable sets the datacenter name of this node. It will set the dc option of cassandra-rackdc.properties. |
| CASSANDRA_RACK | This variable sets the rack name of this node. It will set the rack option of cassandra-rackdc.properties. |
| CASSANDRA_ENDPOINT_SNITCH | This variable sets the snitch implementation this node will use. It will set the endpoint_snitch option of cassandra.yml. |

## <a name="references-volumes"></a>Volumes

These are the filesystem paths used by the container image.

| **Path** | **Description** |
|:---------|:----------------|
| /var/lib/cassandra | All Cassandra files are installed here. |
