# kubernetes

Kubernetes is a container orchestration platform

## problems using Docker

1. single host nature
2. No Auto healing
3. No Auto Scaling
4. No enterprise level support

> Docker is never used in production
> The Above All problems is solved by Kubernetes

By default Kubernetes is a cluster. cluster is basically group of Nodes. Kubernetes has replica sets. Even before the container is going down , kubernetes will start the new container thats is Kubernetes controls the damage 

## k8s Architecture

1. in kubernetes `worker node` or `data-plane` there are three components:
   * `kube-proxy` is responsible for networking like generating ip address. It uses Ip tables for the configuration of networking
   * `kubelet` is responsible for creation of pod, it ensures that pod is always in running state
   * `container runtime` is responsible for running the container

2. Kubernetes `master node` or `control-plane` components: 
   * `Api server` is a component that basically exposes kubernetes. The Api server basically takes all the requests from external world.
   * `scheduler` is basically responsible for scheduling your pods or resources on kubernetes
   * `etcd` is basically a key value store and the entire kubernetes cluster information is stored as objects or key value pairs inside this etcd.
   * In kubernetes By default there are multiple controllers like Replica sets and there has to be a component which ensures that this controller are running that component is called `controller manager`
   * `c-c-m (cloud control manager)` is an open source utility . CCM is a critical component in Kubernetes that simplifies the management of cloud-specific resources by providing an abstraction layer between Kubernetes and cloud providers.

## k8s Production Systems

1. Kubernetes distribution provides customer experience. kubernetes distributions in the order they highly popular :
   * Kubernetes
   * openshift
   * Ranctier
   * Tanzu
   * EKS
   * AKS
   * GKE
   * DKE

2. Kops (K8s operations) manages the lifecycle of kubernetes that is install,upgrade, modifications and deletion of cluster.


## Kubernetes Installation Using KOPS on EC2

### Create an EC2 instance or use your personal laptop.

Dependencies required 

1. Python3
2. AWS CLI
3. kubectl

### Install dependencies

```console
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

```console
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
```

```console
sudo apt-get update
sudo apt-get install -y python3-pip apt-transport-https kubectl
```

```console
pip3 install awscli --upgrade
```

```console
export PATH="$PATH:/home/ubuntu/.local/bin/"
```

### Install KOPS

```console
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64

chmod +x kops-linux-amd64

sudo mv kops-linux-amd64 /usr/local/bin/kops
```

### Provide the below permissions to your IAM user. If you are using the admin user, the below permissions are available by default

1. AmazonEC2FullAccess
2. AmazonS3FullAccess
3. IAMFullAccess
4. AmazonVPCFullAccess

### Set up AWS CLI configuration on your EC2 Instance or Laptop.

Run `aws configure`

## Kubernetes Cluster Installation 

Please follow the steps carefully and read each command before executing.

### Create S3 bucket for storing the KOPS objects.

```console
aws s3api create-bucket --bucket kops-ravi-storage --region us-east-1
```

### Create the cluster 

```console
kops create cluster --name=demok8scluster.k8s.local --state=s3://kops-abhi-storage --zones=us-east-1a --node-count=1 --node-size=t2.micro --master-size=t2.micro  --master-volume-size=8 --node-volume-size=8
```

### Important: Edit the configuration as there are multiple resources created which won't fall into the free tier.

```console
kops edit cluster myfirstcluster.k8s.local
```

Build the cluster

```console
kops update cluster demok8scluster.k8s.local --yes
```

This will take a few minutes to create............

After a few mins, run the below command to verify the cluster installation.

```console
kops validate cluster demok8scluster.k8s.local
```

## pod

`pod` is described as a definition of how to run a container. Normally in docker to run a container we run a command like docker run -p -it like that but, in kubernetes we use a pod.yaml file indicate everything , in pod we can have single or multiple containers.

`kubectl` is command line for kubernetes 


## Deploy first App

1. goto to browser and search for kubectl installation -> goto docs -> choose os .

   ```console
   $ kubectl version     # to check kubectl is installed or not 
   ```

2. now install local kubernetes cluster . you can use minikube , k3s , kind , microk8s. but im using minikube.
   * goto to browser and search for `minikube installation` -> goto docs -> choose os.

   ```console
   $ minikube           # to check minikube is installed
   $ minikube start     # to start minikube cluster. It creates 1vm and single node kubernetes cluster. By default it uses `docker driver`.
   $ minikube start --memory=4096 --driver=hyperkit      # if you want to use hyperkit driver use this command. it is used for advanced kubernetes
   $ kubectl get nodes
   ```

3. To create pod goto to browser and search for `kubernetes pod` and copy the desired application code from the examples.

   ```console
   $ vi pod.yml                   # write the code or paste from the examples here and do some changes according to your application
   $ kubectl create -f pod.yml    # it will creates the pod that is our application is created
   $ kubectl get pods             # to check the pods
   $ kubectl get pods -o wide     # to get more details of pods
   $ minikube ssh                 # to go into cluster
   $ curl ip_address_of_pod
   $ kubectl describe pod nginx    # to see the details
   $ kubectl delete pod nginx     # to delete the pod
   $ kubectl get deploy
   $ kubectl delete deploy nginx
   ```

   > reference is goto browser and search for `kubectl cheatsheet`

## kubernetes Deployment

Deployment will create a Replica set and this replica sets create a pod. And this replica set is a kubernetes controller. controller makes sure that the actual state is same as the desired state. 



| Container      | Pod | Deployment     |
| ---        |    ----   | --- |
| to run container we give docker run command       | we write running specifications of docker container pod.yml file. It may single or multiple containers.       | If you deploy the Deployment then Healing and scaling can be done   |

| Deployment             | Replica set     |
|    ----         |   ---          |
| Deployment will create a Replica set           | It is basically a kubernetes controller that is one that is implementing the auto healing feature of pod.      |

1. lets implement it:

   ```console
   $ vi pod.yml
   $ kubectl create -f pod.yml
   $ kubectl get pods
   $ kubectl get all
   $ kubectl get all -A
   $ kubectl get pods -o wide
   
   ```

2. goto browser search for `kubernetes deployment` and see the example docs

   ```console
   $ vim deployment.yml
   $ kubectl apply -f deployment.yml
   $ kubectl get deploy
   $ kubectl get pod
   $ kubectl get rs     # rs means replica set
   ```

3. Now open another terminal and run this command

   ```console
   $ kubectl get pods -w  # you can watch live whats happening
   $ kubectl delete pod pod_name  # run this command in previous terminal
   ```

   > before the pod gets deleted the replica set ensures another pod gets created

## kubernetes service

1. the service can do :
   * Load Balancing
   * Service Discovery(Labels and selectors)
   * expose to world

2. service types:
   * `Cluster IP` - you can only access inside the cluster
   * `Nodeport` - you can access inside the organization means who had the instance ip address 
   * `Loadbalancer` - entire world

> when the pod is created it as a ip address . if the pod is deleted then a new pod gets created but it has a different ip address. the problem is here is if someone is accessing the application in the pod with the ip , we have no idea when the pod deleted and changes and new ip comes so the application in the pod cannot be accessed . to resolve this issue we use service discovery that uses labels and selectors for the ipaddress.


## kubeshark

```console
$ minikube status       # to check cluster is running
$ kubectl get all
$ kubectl delete deploy deployment_name
$ kubectl delete svc demo-service
$ kubectl get all       # now the default kubernetes service should be running 
```

1. lets build a image of an application

   ```console
   $ vim Dockerfile           # write your application 
   $ docker build -t ravteja/app:V1      # image is build

   ```

2. lets do the deployment

   ```console
   $ vim deployment.yml       # goto `kubernetes deployment` and copy the code paste in this file and do necessary changes , labels are important
   $ kubectl apply -f deployment.yml
   $ kubectl get deploy       # gives the status 
   $ kubectl get pods         # shows pods information
   $ kubectl get pods -o wide  # to get ip address
   ```

3. lets create the service

   ```console
   $ vim service.yml         # goto `kubernetes service` and copy the code paste in this file and do necessary changes, use node-port for now. make sure that the label name in deployment.yml file matches with the selector in service.yml file.
   $ kubectl apply -f service.yml
   $ kubectl get svc         # now copy the ip of the instance and paste in the browser public-ip:port_number
   $ kubectl edit svc svc_name
   ```

## Ingress

1. goto browser search for `kubernetes ingress` and see the example docs

2. lets write the ingress

   ```console
   $ ingress.yml      # replace the service name, and remember the host name
   $ kubectl apply -f ingress.yml
   $ kubectl get ingress      # you cant see nothing in the address because you havent created ingress controller
   ```

3. goto browser search for `kubernetes ingress install` , lets install nginx ingress fo now

   ```console
   $ minikube addons enable ingress
   $ kubectl get ingress        # now you can find in the address
   ```