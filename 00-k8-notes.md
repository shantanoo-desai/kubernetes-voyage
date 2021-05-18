# Kubernetes Locally

## What is Kubernetes?

> Kubernetes is a production-grade, open-source platform that orchestrates the placement (scheduling) and execution of application containers within and across computer clusters.

### Kubernetes Resources

1. Control Plane : Co-ordinates the cluster
2. Nodes: workers that run applications. Node can a physical computer / Virtual Machine.

### How does Communication work between Control Plane & Worker Nodes

- Nodes have a **kubelet** agent for managing themselves and communicating with Control Plane.
- Control Plane and Nodes communicate using the **Kubernetes API**



## Specifications

### OS
Windows 10 Pro Version 2004 OS-build 19041.928

### Docker Desktop Version

Version 3.3.1 (63152)

Engine - 20.10.5
Compose - 1.29.0
Kubernetes - v1.19.7

> **Note**: if using Docker Desktop enable Kubernetes from the GUI and skip past the `minikube` setup.
> Also replace `kubectl.exe` to `kubectl` in WSL2 shell terminal


### minikube

#### Setup

Open PowerShell in Administrative Mode and execute:

```shell
> minikube start --memory=4400mb
* minikube v1.19.0 auf Microsoft Windows 10 Pro 10.0.19041 Build 19041
* Using the hyperv driver based on existing profile
! You cannot change the memory size for an exiting minikube cluster. Please first delete the cluster.
* Starting control plane node minikube in cluster minikube
* Restarting existing hyperv VM for "minikube" ...
* Vorbereiten von Kubernetes v1.20.2 auf Docker 20.10.4...
* Verifying Kubernetes components...
  - Using image gcr.io/k8s-minikube/storage-provisioner:v5
* Enabled addons: storage-provisioner, default-storageclass
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```


```bash
$ minikube.exe version

minikube version: v1.19.0
commit: 15cede53bdc5fe242228853e737333b09d4336b5
```

### kubectl

`kubectl` command-line tool that interacts with the cluster via the **Kubernetes API** 

#### Syntax

> kubectl _action_ _resource_ _options_

`<action>`: create, expose, run ...

```bash
$ kubectl.exe version

des@IKAP-TOP28:~/kubernetes$ kubectl.exe version
Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.7", GitCommit:"1dd5338295409edcfff11505e7bb246f0d325d15", GitTreeState:"clean", BuildDate:"2021-01-13T13:23:52Z", GoVersion:"go1.15.5", Com
piler:"gc", Platform:"windows/amd64"}
Server Version: version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.2", GitCommit:"faecb196815e248d3ecfb03c680a4507229c2a56", GitTreeState:"clean", BuildDate:"2021-01-13T13:20:00Z", GoVersion:"go1.15.5", Com
piler:"gc", Platform:"linux/amd64"}
```
### Cluster-Info

View the Cluster

```bash
$ kubectl.exe cluster-info

$ kubectl cluster-info
Kubernetes master is running at https://kubernetes.docker.internal:6443
KubeDNS is running at https://kubernetes.docker.internal:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```
### Nodes

Show Nodes that can host our applications

```bash
$ kubectl.exe get nodes

NAME       STATUS   ROLES                  AGE   VERSION
minikube   Ready    control-plane,master   23d   v1.20.2
```

### Deployment

used for creating and updating instances of your applications. **Kubernetes Deployment Controller** monitors the instances.

If application goes down/deleted, **Kubernetes Deployment Controller** replaces the instance on another node in cluster.

#### Create a Deployment

```bash
$ kubectl.exe create deployment <deployment_name> --image=<URL_of_Container_Image>

```
Example:

```bash
$ kubectl.exe create deployment shan-k8s-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
deployment.apps/shan-k8s-bootcamp created
```

#### Get a Deployment

```bash
$ kubectl.exe get deployments
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
shan-k8s-bootcamp   1/1     1            1           3m39s
```

> Note: Pods running inside a k8s cluster are on a private network and not accessible directly.

### Kubernetes Proxy

create a proxy to forward communications into the cluster's private network. API server willl create an endpoint for each
**Pod** based on the Pod name and make it accessible via the proxy

```bash
$ kubectl.exe proxy
Starting to serve on 127.0.0.1:8001
```

Example: check with a browser: http://localhost:8001/version

```json
{
  "major": "1",
  "minor": "20",
  "gitVersion": "v1.20.2",
  "gitCommit": "faecb196815e248d3ecfb03c680a4507229c2a56",
  "gitTreeState": "clean",
  "buildDate": "2021-01-13T13:20:00Z",
  "goVersion": "go1.15.5",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

### Kubernetes Pods

Pod represents a group of one or more containers and some _shared resources_ between them.

Shared resources:

1. Shared Storage (volumes)
2. Networking (unique Cluster IP address)
3. Information about running the container (image version, specific ports)

Containers in a Pod share an IP Address and Port Space

> Pods are the atomic unit of Kubernetes.

#### How it ties up with Deployments?

    User -- Create --> Deployment --> API Server -- Create --> Pods ( Containers ...)
 

#### Get Information about the Pod

```bash
$ kubectl.exe get pods
NAME                                 READY   STATUS    RESTARTS   AGE
shan-k8s-bootcamp-66ff8484b5-78z8l   1/1     Running   0          13m

$ kubectl.exe get pods -o json # get detail in JSON format
$
$ kubectl.exe describe pods # get extensive detail of pod in Human-Readable format
```

> Pod Name Syntax: `DeploymentName-PodGeneratedHash-RandomGeneratedName`


### Kubernetes Nodes

Node: Worker machine (virtual / physical) 

Each Node is managed by Kubernetes Control Plane (responsible for scheduling Pods on Nodes)

Every Node runs at least:

- Kubelet: communication between k8s master and node, manages running container on the node
- Container Runtime: for pulling images from a registry, unpacking the container, running the app

> Pods have a _lifetime_ and are _mortal_.

Each Pod in a Node has unique IP Address, even for pods with same names.

### Kubernetes ReplicaSets

Maintain a replicas of the Pods on a Node, in order to avoid downtimes. Just keep copies running in order to avoid a Pod failure.

#### Get ReplicaSets

```bash
$ kubectl.exe get rs
```

> Syntax: [_deployment-name]-[_random-string-using-pod-template-hash-as-seed]

_Example_:

```bash
$ kubectl.exe get rs
NAME                             DESIRED   CURRENT   READY   AGE
kubernetes-bootcamp-57978f5f5d   1         1         1       4m35s
```

- `DESIRED`: desired number of replicas of the application
- `CURRENT`: how many replicas are currently running

### Kubernetes Service

Abstraction layer to define _logical set_ of Pods and enable:
- **External Traffic Exposure**
- **Load-Balancing**
- **Service Discovery of Pods**

Defined using `json` or `yaml` file

Set of Pods targeted by a service is determined by `LabelSelector`

Service allows your application to receive traffic

#### Exposing Service

by specifying `type` in `ServiceSpec`

These `type` are:

- `ClusterIP`: expose service on internal IP in the cluster. only reachable within cluster
- `NodePort`: expose service on `<NodeIp>:<NodePort>` using NAT
- `LoadBalancer`: external Load Balancer and assigns a fixed IP to Service.
- `ExternalName`: mapping to `CNAME` for DNS


    LoadBalancer::Superset(NodePort::SuperSet(ClusterIP))


#### Get Services

```bash
$ kubectl.exe get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   24d
```

#### Expose Service

```bash
$ kubectl.exe expose deployment/<deployment_name> --type="NodePort" --port 8080
```

_Example_

```bash
$ kubectl.exe expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes            ClusterIP   10.96.0.1       <none>        443/TCP          24d
kubernetes-bootcamp   NodePort    10.101.57.158   <none>        8080:32066/TCP   7s
```

here `NodePort` will be `32066` for external usage


#### Get Description of Services

```bash
$ kubectl.exe describe services/<name_of_service>
```

_Example_

```bash
$ kubectl.exe describe services/kubernetes-bootcamp
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   app=kubernetes-bootcamp
Annotations:              <none>
Selector:                 app=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.101.57.158
LoadBalancer Ingress:     localhost
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  32066/TCP
Endpoints:                10.1.0.26:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

Get the `minikube` IP Address using:

```shell
> minikube ip
```

and check using `http://<MINIKUBE_IP>:NodePort`


#### Service Labels

Deployment creates a Label for us:

```bash
$ kubectl describe deployments | grep "Labels"
  Labels:  app=kubernetes-bootcamp
```

Access Pods/Services using this Service Label:

```bash
$ kubectl.exe get pods -l app=kubernetes-bootcamp
$ kubectl.exe get services -l app=kubernetes-bootcamp
```

#### Apply a new Label

```bash
$ kubectl.exe label pod <POD_NAME> <LabelName>
```

_Example_

```bash
$ kubectl.exe label pod kubernetes-bootcamp-57978f5f5d-nptrt version=v1
```

To see the applied label, get pod description

```bash
$ kubectl.exe describe pods <POD_NAME>
```

#### Use Labels 

```bash
$ kubectl.exe get pods -l <LabelName> # here version=v1
```

#### Delete Service

```bash
$ kubectl.exe delete service -l <LabelName>
```

##Scaling using Kubernetes

When traffic increases to app, you need to scale

**Scaling** accomplished by changing replicas in Deployment

Distributing traffic to such replicas of the same application
on different Nodes needs a **load-balancer** provided by **Kubernetes Services**

```bash
$ kubectl.exe scale deployments/<deployment-name> --replicas=Replica-Numbers
```

_Example_

```bash
$ kubectl.exe scale deployments/kubernetes-bootcamp --replicas=4
deployment.apps/kubernetes-bootcamp scaled
```

Get more information on scaled deployment using:

```bash
$ kubectl.exe get pods -o wide

NAME                                   READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
kubernetes-bootcamp-57978f5f5d-k579l   1/1     Running   0          5s    172.17.0.6   minikube   <none>           <none>
kubernetes-bootcamp-57978f5f5d-k6ns2   1/1     Running   0          15m   172.17.0.3   minikube   <none>           <none>
kubernetes-bootcamp-57978f5f5d-m7gph   1/1     Running   0          5s    172.17.0.4   minikube   <none>           <none>
kubernetes-bootcamp-57978f5f5d-xkdrl   1/1     Running   0          5s    172.17.0.5   minikube   <none>           <none>
```

Note new pods with different IP Addresses are now _scaled up_ with the application.

Scaling Up is logged into the deployment which is visible via different IP addresses
for the Pods

```bash
$ kubectl.exe describe deployments/kubernetes-bootcamp
```

#### Load-Balancing for Replicas

Deploy a load-balancer service using

```bash
$ kubectl.exe expose deployments/kubernetes-bootcamp \
  --type="LoadBalancer" \
  --port=8080
```

Get the `NodePort` using:

```bash
$ kubectl.exe get services/kubernetes-bootcamp
```

Use **PostMan** to query the application on the following

    MinikubeIP:NodePort

> __NOTE__: make sure to disable `Connection` in the header settings of query

This is should hit different pods each pods on every query.

You can verify this by checking the name of the Pods and the payload for the query
in this particular case of `kubernetes-bootcamp:v1` application

#### Scaling Down

Scale Down by reducing the number of replicas for the application

__Example__

```bash
$ kubectl.exe scale deployments/kubernetes-bootcamp --replicas=2
```
Reduce from 4 -> 2 in this case

### Rolling Updats in Kubernetes

Deploy new versions of application without downtimes

Achieved via incrementatlly updating Pods instances with new ones

> by default, maximum number of pods that can be unavailable during the update
  and maximum number of pods that can be created is ONE

However, can be configured to be numbers or percentages of Pods.

> If Deployment exposed publicly, Service will load-balance the traffic only to 
 available Pods during the update


#### Update Application

```bash
$ kubectl.exe set image deployments/DEPLOYMENT-NAME \ 
  DEPLOYMENT-NAME=ContainerRepository:Version
```

_Example_

```bash
$ kubectl.exe set image deployments/kubernetes-bootcamp \
  kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
```

#### Rollout status

```bash
$ kubectl.exe rollout status deployments/DEPLOYMENT-NAME
```

_Example_

```bash
$ kubectl.exe rollout status deployment/kubernetes-bootcamp
deployment "kubernetes-bootcamp" successfully rolled out
```

#### Undo Rollout

If you added some false version e.g. `kubernetes-bootcamp:v10` then you can revert
back to the previous version using

```bash
$ kubectl.exe rollout undo deployments/kubernetes-bootcamp
```

### Applying Configuration

```bash
$ kubectl apply -f DEPLOYMENT_CONFIGURATION.yml
```

_Example_:

```bash
$ kubectl apply -f configurations/01-nginx-deployment.yml
```

You can change the configuration and apply `kubectl apply -f config.yaml` and changes are picked up

`-f` flag determines the file for the deployment
