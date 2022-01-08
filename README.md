## Install Kubernetes on Macos (Without homebrew)
- Download the latest release:
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl" 
```

- Validate the binary (optional)
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl.sha256"
```

- Make the kubectl binary executable.
```
chmod +x ./kubectl
```

- Move the kubectl binary to a file location on your system PATH.
```
sudo mv ./kubectl /usr/local/bin/kubectl
sudo chown root: /usr/local/bin/kubectl
```

- Test to ensure the version you installed is up-to-date:
```
kubectl version --client
```

### Install with Homebrew on macOS
If you are on macOS and using Homebrew package manager, you can install kubectl with Homebrew.

- Run the installation command:
```
brew install kubectl 
or
brew install kubernetes-cli
```

- Test to ensure the version you installed is up-to-date:
```
kubectl version --client
```

For more follow

https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/


### Setup and configure doctl(DigitalOcean CLI)
-  Install doctl
```
brew install doctl
```

- Create an API token
  Create a DigitalOcean API token for your account with read and write access from the Applications & API page in the control panel. The token string is only displayed once, so save it in a safe place. 
  https://cloud.digitalocean.com/account/api/tokens
  
- Use the API token to grant account access to doctl
Use the API token to grant doctl access to your DigitalOcean account. Pass in the token string when prompted by doctl auth init, and give this authentication context a name.

```
doctl auth init --context <NAME>
```

Authentication contexts let you switch between multiple authenticated accounts. You can repeat steps 2 and 3 to add other DigitalOcean accounts, then list and switch between authentication contexts:

```
doctl auth list
doctl auth switch --context <NAME>
```

- Validate that doctl is working
```
doctl account get
```

### Creating a Kubernetes Cluster on DigitalOcean and using that for Deploying Message Queue
We can create kubernetes cluster by different ways 
- using doctl cli
- using DO GUI and dashboard


- Go to Kubernetes Section in DigitalOcean dashboard
https://cloud.digitalocean.com/kubernetes/clusters
<img width="1439" alt="Screenshot 2022-01-08 at 5 47 07 PM" src="https://user-images.githubusercontent.com/15937992/148643824-2bd4a420-1047-49d1-a091-8dc13b95b0fe.png">

- Click on create Cluster and make required changes, create cluster
<img width="1440" alt="Screenshot 2022-01-08 at 5 48 44 PM" src="https://user-images.githubusercontent.com/15937992/148643862-f0519ca8-38c2-4b43-ba67-90183c646b8d.png">

- After creating cluster go to overview and follow instructions
<img width="1238" alt="Screenshot 2022-01-08 at 5 49 27 PM" src="https://user-images.githubusercontent.com/15937992/148643875-5a252814-d860-4527-bf0c-4934606da21c.png">

- Use below command(Step two on above overview)
```
doctl kubernetes cluster kubeconfig save <CONTEXT>
```

This command will connect digitalocean cluster with your local command line and you can perform any actions directly using `kubectl` command on CLI.

- After setting up cluster use below commands to setup message queue setup on kubernetes cluster.



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
