# RabbitMQ cluster
This project will guid you to create your own RabbitMQ cluster and test its performance.

## Prerequisites

```shell
docker pull gitlab-registry.cern.ch/cmsos/k8sbox/rabbitmq/cmsos-x86_64-rabbitmq:0.0.0.5
```

# Deploy RabbitMQ cluster using Kubernetes

## Manual

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

### Run measurement cleanup
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

## Helm charts

- You can adjust `values.yaml` file in both `rabbitmq-cluster-operator` and `benchmark` directory to reflect your needs.

```shell
helm install rabbitmq-operator rabbitmq-cluster-operator/
helm install benchmark benchmark/
```

### Remove all deployed pods
```shell
helm uninstall rabbitmq-operator
helm uninstall benchmark 
```