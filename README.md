# KEDA-rabbitmq-microservices

In this project `Event Driven` archutecture is demonstrated through a producer consumer model is created with RabbitMQ queue and Autoscale via KEDA.  The consumer will receive a single message at a time (per pod), and sleep for 1 second to simulate performing work.  When adding a massive amount of queue messages, KEDA will drive the container to scale out according to the event source present in RabbitMQ(basically messages insdie queue).

## Pre-requisites

* Kubernetes cluster
* KEDA installed

## Install KEDA uisng Helm

```
helm repo add kedacore https://kedacore.github.io/charts
helm repo update
helm install keda kedacore/keda --namespace keda --create-namespace
```

## Getting Started

This setup will go through creating a RabbitMQ queue on the cluster and deploying this consumer with the `ScaledObject` to scale via KEDA. If you already have RabbitMQ you can use your existing queues.

First you should clone the project:

```
git clone https://github.com/satyazzz123/KEDA-rabbitmq-microservices.git

```

#### Install RabbitMQ via Helm



```
helm repo add bitnami https://charts.bitnami.com/bitnami

helm install rabbitmq --set auth.username=user --set auth.password=PASSWORD bitnami/rabbitmq --wait
```


* If you are running the rabbitMQ image on KinD, you will run into permission issues unless you set `volumePermissions.enabled=true`. Use the following command if you are using KinD:

    ```
    helm install rabbitmq --set auth.username=user --set auth.password=PASSWORD --set volumePermissions.enabled=true bitnami/rabbitmq --wait
    ```


### Deploying the RabbitMQ consumer

#### Deploy a consumer

```
kubectl apply -f deploy/deploy-consumer.yaml
```

#### Validate the consumer has deployed

![alt text](<Screenshot from 2024-09-19 14-02-25.png>)

#### Rabbitmq UI (to see it kubectl port-forward service/rabbitmq 3000:15672)
![image](https://github.com/user-attachments/assets/9b884be2-0338-44a8-abc2-72eca56d0d0b)



### Publishing messages to the queue

#### Deploy the publisher job

The following job will publish 300 messages to the "hello" queue the deployment is listening to. As the queue builds up, KEDA will help the horizontal pod autoscaler add more and more pods until the queue is drained after about 2 minutes and up to 30 concurrent pods.  You can modify the exact number of published messages in the `deploy-publisher-job.yaml` file.

```
kubectl apply -f deploy/deploy-publisher-job.yaml
```

#### Validate the deployment scales

```
watch kubectl get all
```

You can watch the pods spin up and start to process queue messages.  As the message length continues to increase, more pods will be pro-actively added. 

![alt text](<Screenshot from 2024-09-19 14-01-33.png>)


After the queue is empty and the specified cooldown period (a property of the `ScaledObject`, default of 300 seconds) the last replica will scale back down to zero.
![alt text](<Screenshot from 2024-09-19 14-02-58.png>)


