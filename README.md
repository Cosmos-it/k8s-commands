# k8s-commands

This is a collection of notes from the internet on how to do things kubernetes

## Most common ways people install Kubernetes

- Minikube: For local developement
- kops and Kubeadm: For clouds and bare metal

General Management Tools

- Dashboard: Visualization
- Kubefed: For Larger cluster federation
- Kompose: For Exporting from docker with compose
- Helm: For Large application management

## Creating a cluster:

When creating a cluster from an image, you will need to use the run command.

- `kubectl run give-name-of-cluster --image repo-name/image-name:latest`

Common way (from file): To use a yaml file within your local computer, use the command

- `kubectl create -f file-2-deploy.yaml`

Multiple files:

- `kubectl create -f file-1-deploy.yaml file-2-deploy.yaml`

Whole directory: Must have .yaml files

- `kubectl create ./cluters`

URL:

- `kubectl apply -f https://github.com/Cosmos-it/k8s-commands/blob/master/sample.yaml`

## Using namespaces

### Working with many applications

Deploy sample infrastructure:

- `kubectl apply -f sample-infrastructure.yaml`

#### Namespaces

- `kubectl get pods` returns no results.
- `kubectl get pods --all-namespaces` returns all our results.
- `kubectl get namespaces` Returns just the namespaces

## Searching, sorting and filtering applications on kubernetes

#### More details

- `kubectl get pods --sort-by=.metadata.name` sorts pods by name
- `kubectl get pods -o wide --all-namespaces` returns more details
- `kubectl get pods/cart-dev -n cart -o json` returns the pod json
- `kubectl get pods -n cart -o=jsonpath="{..image}" -l app=cart-dev` searches cart-dev, and returns the image based on the jsonpath
- `kubectl get pods --all-namespaces -o jsonpath="{.items[*]- .spec.containers[*].image}"` all container images running

## Understand how to delete pods, and resources in k8s

- `kubectl delete pods --all` deletes all pods in the default namespace
- `kubectl delete pods -n cart --all` deletes all pods in the cart namespace
- `kubectl delete pods -l env=staging -n social` Deletes the pods in the social namespace that match the staging environment

## Kops with AWS

To use kops to run your own kubernetes cluster, you will need to the following.

Install and set

- kops
- kubectl
- AWS credentials
- AWS CLI tools

## AWS requirements

- Set up IAM account: Give ut full access to s3, EC2, Route53, and IAM
- Create DNS record for the kubernetes services

### Setup a cluster

- `export NAME=mycluster.k8s.local`
- `export KOPS_STATE_STORE=s3://prefix-example-com-state-store`

## Build cluster

- `kops create cluster \ --zones use-west-2b \ ${NAME}`

Note: Kops create cluster command will result into a configuration file for what your cluster is going tro look like

Important: Important to verify that everything is the way kops expects.

- `kops edit cluster cluster-name`

## Kops update command

- `kops update cluster cluster-name`

Build the cluster per spec

## Check to see if everything is good.

- `kubectl get nodes` alert when nodes are ready
- `kops validate cluster` verify that everything works
- You can use terraform to provison that nodes. (Read more terraform)

# Next on...

### Microservices

Microservices: An oriented architecture that structures the entire application as a collection of loosely coupled services.
