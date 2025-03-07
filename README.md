# RabbitMQ cluster
This project will guid you to create your own RabbitMQ cluster and test its performance.

# Deploy RabbitMQ cluster using Kubernetes
```shell
kubectl apply -f rabbitmq_operator.yaml
kubectl apply -f rabbitmq_storage.yaml
kubectl annotate storageclass local-path storageclass.kubernetes.io/is-default-class=true
kubectl apply -f rabbitmq_cluster.yaml
```

# Show credentials (username & password)
```shell
echo $(kubectl get secret rabbit-cluster-default-user -o jsonpath="{.data.username}" | base64 --decode)
echo $(kubectl get secret rabbit-cluster-default-user -o jsonpath="{.data.password}" | base64 --decode)

```

# Run measurement 
- Using [MQTT Benchmark](https://github.com/danyk20/MQTT_Benchmark) 
```shell
kubectl apply -f consumers_jobs.yaml
kubectl apply -f publishers_jobs.yaml
```