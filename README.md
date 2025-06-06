# RabbitMQ cluster

This project will guide you to test performance by MQTT Benchmark tool and create your own RabbitMQ cluster using
individual Kubernetes components or Helm chart.

## RabbitMQ Operator

The RabbitMQ Operator is a Kubernetes controller that simplifies the deployment and management of **RabbitMQ** clusters.
It automates tasks like cluster creation, scaling, and configuration through a custom resource `RabbitmqCluster`. The
operator runs in the `rabbitmq-system` namespace and handles **StatefulSet** management, service creation, and secret
handling for credentials. It ensures proper cluster formation with features like peer discovery, persistence, and TLS
configuration. The operator also monitors cluster health and provides status conditions to track readiness. `RBAC` rules
grant it necessary permissions to manage RabbitMQ resources while maintaining security isolation. The deployment uses a
single replica with defined resource limits for reliable operation.

## Configuration of the cluster

The provided configuration defines a highly available RabbitMQ cluster with three replicas for fault tolerance. It uses
the `rabbitmq:3.12-management` image with the MQTT plugin enabled, exposing ports for AMQP `5672`, MQTT `1883`, and
management `15672` via NodePort services. The built-in management UI is accessible via NodePort `31672`. The cluster is
configured with durable quorum queues and debug-level logging. Credentials are managed securely via a Kubernetes Secret
`user`:`password`, and the setup includes custom resource limits (defaulting to 2 CPU / 2GB memory). The cluster
supports persistent storage (10Gi by default). Additional RabbitMQ configurations can be injected via additionalConfig,
while anti-affinity rules help distribute pods across nodes for resilience. The cluster is configured with a **strict
node affinity**, meaning pods will only be scheduled on nodes with the following specific hostnames:

* `d3vrubu-c2e34-10-01`
* `d3vrubu-c2e34-12-01`
* `d3vrubu-c2e34-14-01`
* `d3vrubu-c2e34-16-01`
* `d3vrubu-c2e34-18-01`
* `d3vrubu-c2e34-29-01`
* `d3vrubu-c2e34-31-01`
* `d3vrubu-c2e34-33-01`
* `d3vrubu-c2e34-35-01`

### Helm

Individual Kubernetes components are mapped 1:1 to Helm charts `/benchmark` and `/rabbit-cluster-operator`.

## MQTT benchmark configuration (`producers_jobs.yaml` and `consumers_jobs.yaml`)

The producer and consumer jobs are designed to benchmark RabbitMQ performance under load. The producer job runs two
parallel instances, each sending 10 million fixed-size messages (72 bytes) at maximum speed `--period 0` with QoS 1 for
reliable delivery. The consumer job runs a single instance with QoS 1, processing the last 80% of the 100-second test
duration. Both jobs use node affinity (the same as above) and pod anti-affinity to distribute workloads across different
nodes, avoiding resource contention. They connect to RabbitMQ via NodePort `31883` (MQTT protocol) using credentials
from the `credentials` Secret and store benchmark data in a hostPath volume `~/data`. The jobs leverage a custom Docker
image `cmsos-x86_64-rabbitmq:0.0.1.3` containing benchmarking tools.

## Prerequisites

```shell
docker pull gitlab-registry.cern.ch/cmsos/k8sbox/rabbitmq/cmsos-x86_64-rabbitmq:0.0.1.3
```

# Deploy RabbitMQ cluster using Kubernetes

- Cluster is required to have an odd number of replicas, and a majority of them must be functional at any given time
- Individual running pods will form a hierarchy with one leader and the others are followers
- Using quorum ques is recommended to achieve high-reliability and consistency guarantees
    - quorum queues are replicated by design
    - based on the Raft consensus algorithm, ensuring that messages are replicated across multiple nodes before being
      acknowledged—this prevents data loss even if a node fails
    - leader election is automatically using Raft—if the leader node fails, a new leader is elected seamlessly

## Manual deployment of individual Kubernetes components

### RabbitMQ cluster deployment

```shell
kubectl apply -f rabbitmq_operator.yaml
kubectl apply -f rabbitmq_storage.yaml
kubectl apply -f rabbitmq_secret.yaml
kubectl annotate storageclass local-path storageclass.kubernetes.io/is-default-class=true
kubectl apply -f rabbitmq_definitions.yaml
kubectl apply -f rabbitmq_cluster.yaml
```

### Run measurement using the MQTT benchmark tool

- Using [MQTT Benchmark](https://github.com/danyk20/MQTT_Benchmark)

```shell
kubectl apply -f consumers_jobs.yaml
kubectl apply -f producers_jobs.yaml
```

### Measurement cleanup

```shell
kubectl delete -f consumers_jobs.yaml
kubectl delete -f producers_jobs.yaml
```

### RabbitMQ cluster cleanup

```shell
kubectl delete -f rabbitmq_definitions.yaml
kubectl delete -f rabbitmq_cluster.yaml
kubectl delete -f rabbitmq_secret.yaml
kubectl delete -f rabbitmq_storage.yaml
kubectl delete -f rabbitmq_operator.yaml
```

## Automated deployment using Helm charts

- You can adjust `values.yaml` file in both `rabbitmq-cluster-operator` and `benchmark` directory to reflect your needs.

```shell
helm install rabbitmq-operator rabbitmq-cluster-operator/
helm install benchmark benchmark/
```

### Remove all deployed pods

```shell
helm uninstall benchmark 
helm uninstall rabbitmq-operator
```