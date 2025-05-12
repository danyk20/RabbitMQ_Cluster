# RabbitMQ cluster

This project will guide you to create your own RabbitMQ cluster and test its performance using MQTT benchmark tool.

## Prerequisites

```shell
docker pull gitlab-registry.cern.ch/cmsos/k8sbox/rabbitmq/cmsos-x86_64-rabbitmq:0.0.1.0
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
kubectl apply -f rabbitmq_cluster.yaml
```

### Run measurement

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