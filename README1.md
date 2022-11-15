- Create an EC2 Machine for Master Node of Cluster (4GB RAM, 30GB Storage, Ubuntu 20)

- Change hostname of EC2 machine to something more meaningful
  ```
  sudo nano /etc/hostname
  sudo nano /etc/hosts
  sudo reboot
  ```

- Install aws cli
  - Follow this (https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
  ```
  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  
  unzip awscliv2.zip
  
  sudo ./aws/install
  
  which aws
  
  aws configure
  ```

- Install kubectl
  - https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
  
  ```
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

    chmod +x kubectl

    mkdir -p ~/.local/bin

    mv ./kubectl ~/.local/bin/kubectl

    kubectl version --client
  ```
  
- Create IAM programmatic user and a role with IAM, EC2, Route53 and S3 full access, and download access and secret key.
 
- Configure aws in ubuntu or just allocate role to EC2 instance that you just created above.
 
- Install KOps (https://kubernetes.io/docs/setup/production-environment/tools/kops/)
    ```
    curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    chmod +x kops-linux-amd64
    sudo mv kops-linux-amd64 /usr/local/bin/kops
    ```
  
- Register a Domain in Route53 and create a public hosted zone
 
- create a s3 bucket
    ```
    aws s3 mb s3://dev.k8s.codesail.link
    ```
  
- Expose environment variable
    ```
    export KOPS_STATE_STORE=s3://dev.k8s.codesail.link
    ```
  
- Create ssh keys before creating cluster
  - ssh-keygen
 
- Create kubernetes cluster

  ```
  export KOPS_STATE_STORE=s3://dev.k8s.codesail.link
  export MASTER_SIZE=t2.medium
  export NODE_SIZE=t2.small
  
  kops create cluster --cloud=aws --zones=ap-south-1 --name=dev.k8s.codesail.link --dns-zone=codesail.link --dns public/private
  
  kops update cluster dev.k8s.codesail.link --yes
  ```
  
  https://kops.sigs.k8s.io/cli/kops_create_cluster/
 
