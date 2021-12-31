### Applying Strimzi installation file
Before deploying Strimzi Kafka operator, let’s first create our kafka namespace:

### kubectl create namespace kafka
Next we apply the Strimzi install files, including ClusterRoles, ClusterRoleBindings and some Custom Resource Definitions (CRDs). The CRDs define the schemas used for declarative management of the Kafka cluster, Kafka topics and users.

```kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka```
This will be familiar if you’ve installed Strimzi on things like minikube before. Note how we set all the namespace references in downloaded .yaml file to kafka. By default they are set to myproject. But we want them all to be kafka because we decided to install the operator into the kafka namespace, which we achieve by specifying -n kafka when running kubectl create ensuring that all the definitions and the configurations are installed in kafka namespace rather than the default namespace.

If there is a mismatch between namespaces, then the Strimzi Cluster Operator will not have the necessary permissions to perform its operations.

Follow the deployment of the Strimzi Kafka operator:

```kubectl get pod -n kafka --watch```
You can also follow the operator’s log:

kubectl logs deployment/strimzi-cluster-operator -n kafka -f
Provision the Apache Kafka cluster
Then we create a new Kafka custom resource, which will give us a small persistent Apache Kafka Cluster with one node for each - Apache Zookeeper and Apache Kafka:

# Apply the `Kafka` Cluster CR file
```kubectl apply -f https://strimzi.io/examples/latest/kafka/kafka-persistent-single.yaml -n kafka``` 
We now need to wait while Kubernetes starts the required pods, services and so on:

```kubectl wait kafka/my-cluster --for=condition=Ready --timeout=300s -n kafka```
The above command might timeout if you’re downloading images over a slow connection. If that happens you can always run it again.

Send and receive messages
Once the cluster is running, you can run a simple producer to send messages to a Kafka topic (the topic will be automatically created):

```kubectl -n kafka run kafka-producer -ti --image=quay.io/strimzi/kafka:0.27.0-kafka-3.0.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic my-topic```
And to receive them in a different terminal you can run:

```kubectl -n kafka run kafka-consumer -ti --image=quay.io/strimzi/kafka:0.27.0-kafka-3.0.0 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning```
Enjoy your Apache Kafka cluster, running on Kind!
