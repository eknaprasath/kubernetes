ETCD - PORT -2379

to check the available keys

kubectl exec etcd-master -n kube-system etcdctl get / --prefix -keys-only

ETCD - Commands (Optional)
(Optional) Additional information about ETCDCTL Utility

ETCDCTL is the CLI tool used to interact with ETCD.

ETCDCTL can interact with ETCD Server using 2 API versions - Version 2 and Version 3.  By default its set to use Version 2. Each version has different sets of commands.

For example ETCDCTL version 2 supports the following commands:

etcdctl backup
etcdctl cluster-health
etcdctl mk
etcdctl mkdir
etcdctl set


Whereas the commands are different in version 3

etcdctl snapshot save 
etcdctl endpoint health
etcdctl get
etcdctl put

To set the right version of API set the environment variable ETCDCTL_API command

export ETCDCTL_API=3



When API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don't work. When API version is set to version 3, version 2 commands listed above don't work.



Apart from that, you must also specify path to certificate files so that ETCDCTL can authenticate to the ETCD API Server. The certificate files are available in the etcd-master at the following path. We discuss more about certificates in the security section of this course. So don't worry if this looks complex:

--cacert /etc/kubernetes/pki/etcd/ca.crt     
--cert /etc/kubernetes/pki/etcd/server.crt     
--key /etc/kubernetes/pki/etcd/server.key


So for the commands I showed in the previous video to work you must specify the ETCDCTL API version and path to certificate files. Below is the final form:



kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key" 


##KUBEAPI

kubectl get nodes

you run a kubectl command, the kubectl utility is infact reaching to the kube-apiserver.
The kube-api server first authenticates the request and validates it. It then retrieves the data from the
ETCD cluster and responds back with the requested information.


##CREATION OF POD
1. Request authenticate user
2. Validate Request
3. Retrieve data
4. Update ETCD
5. Scueduler
6. Kubelet


example of creating a pod when you do that as before the request is authenticated first and then validated. In this case, the API server creates a POD object without assigning it to a node, updates the information in the ETCD server updates the user that the POD has been created. The scheduler continuously monitors the API server and realizes that there is a new pod with no node

assigned the scheduler identifies the right node to place the new POD on and communicates that back to the kube-apiserver.The API server then updates the information in the ETCD cluster. The API server then passes that information to the kubelet in appropriate worker node.

The kubelet then creates the POD on the node and instructs the container runtime engine to deploy the application image. Once done, the kubelet updates the status back to the API server and the API server then updates the data back in the ETCD cluster. A similar pattern is followed every time a change is requested. The kube-apiserver is at the center of all the different tasks that needs to be performed to make a change in the cluster


POD DEFINITION FILE LOCATED AT
for kubeadm setup
cat /etc/kubernetes/manifests/kube-apiserver.yaml

non kubeadm setup
cat /etc/systemd/system/kube-apiserver.service



##KUBE CONTROLLER MANAGER


WATCH STATUS
REMEDIATE SITUATION
NODE MONITOR PERIOD = 5 SECONDS
NODE MONITOR GRACE PERIOD = 40 SECONDS
POD EVICTION TIMEOUT = 5 MINUTES

Deployment-controller, Namespace-Controller, Endpoint-Controller, CronJob, Job-Controller, PV-Protection-Controller, Service-Account-Controller, Stateful-Set, ReplicaSet, Node-Controller, PV-Binder-Controller, Replication-controller.

a controller is a process that continuously monitors the state of various components within the system and works towards bringing the whole system to the desired functioning state. For example the node controller is responsible for monitoring the status of the nodes and taking necessary actions to keep the application running. It does that through the kube-api server. The node controller checks the status of the nodes every 5 seconds. That way the node controller can monitor the health of the nodes if it stops receiving heartbeat from a node the node is marked as unreachable but it waits for 40 seconds before marking it unreachable after a node is marked unreachable it gives it five minutes to come back up if it doesn’t, it removes the PODs assigned to that node and provisions them on the healthy ones. If the PODs are part of a replica set the next controller is the replication controller. It is responsible for monitoring the status of replica sets and ensuring that the desired number of PODs are available at all times within the set. If a pod dies it creates another one. Now those were just two examples of controllers. There are many more such controllers available within kubernetes. Whatever concepts we have seen so far in kubernetes such as deployments, Services, namespaces, persistent volumes and whatever intelligence is built into these constructs it is implemented through these various controllers. As you can imagine this is kind of the brain behind a lot of things in kubernetes. Now how do you see these controllers and where are they located in your cluster. They're all packaged into a single process known as kubernetes controller manager. When you install the kubernetes controller manager the different controllers get installed as well. So how do you install and view the kubernetes Controller manager download the kube-controller-manager from the kubernetes release page. Extract it and run it as a service. When you run it as you can see there are a list of options provided this is where you provide additional options to customize your controller. Remember some of the default settings for node controller we discussed earlier such as the node monitor period the grace period and the eviction timeout. These go in here as options. There is an additional option called controllers that you can use to specify which controllers to enable. By default all of them are enabled but you can choose to enable a select few. So in case any of your controllers don't seem to work or exist this would be a good starting point to look at.

DEFINITION FILE LOCATED AT
for kubeadm setup
cat /etc/kubernetes/manifests/kube-controller-manager.yaml

for non-kubeadm setup
cat /etc/systemd/system/kube-controller-manager.service


##KUBE-SCHEDULER
we discussed that the kubernetes scheduler is responsible for scheduling pods on nodes.
remember the scheduler is only responsible for deciding which pod goes on which node. It doesn’t actually place the pod on the nodes.

place the pod on the nodes. That’s the job of the kubelet. The kubelet or the captain on the ship is who creates the pod on the


RESOURCE REQUIREMENTES AND LIMITS
TAINTS AND TOLERATIONS
NODES SELECTORS/AFFINITY

DEFINITION FILE LOCATED AT
for kubeadm setup
cat /etc/kubernetes/manifests/kube-scheduler.yaml



##KUBELET:

 we discussed that the kubelet is like the captain on the ship. They lead all activities on a ship.
 
The kubelet in the kubernetes worker node, registers the node with the kubernetes cluster. When it receives instructions to load a container or a POD on the node, it requests the container run time engine, which may be Docker, to pull the required image and run an instance. The kubelet then continues to monitor the state of the POD and the containers in it and reports to the kube-api server on a timely basis.

Register Node
Create PODs
Monitor Node & PODs



##KUBE PROXY

Within a kubernetes cluster every pod can reach every other pod. This is accomplished by deploying a POD networking solution to the cluster. A POD network is an internal virtual network that spans across all the nodes in the cluster to which all the PODs connect to. Through this network are able to communicate with each other. There are many solutions available for deploying such a network. In this case I have a web application deployed on the first node and a database application deployed on the second. The web app can reach the database, simply by using the IP of the database POD. But there is no guarantee that the IP of the database part will always remain the same. If you've gone through the lecture on services as discussed in the beginners course you must know that a better way for the web application to access the database is using a service. So we create a service to expose the database application across the cluster. The web application can now access the database using the name of the service db. The service also gets an IP address assigned to it whenever a pod tries to reach the service using its IP or name it forwards the traffic to the back end pod. In this case the database. But what is this service and how does it get an IP? Does the service join the same POD Network? The service cannot join the pod network because the service is not an actual thing. It is not a container like pod so it doesn't have any interfaces or an actively listening process. It is a virtual component that only lives in the cabinet as memory. But then we also said that the service should be accessible across the cluster from any not. So how is that achieved? That’s where kube-proxy comes in. Kube-proxy is a process that runs on each node in the kubernetes cluster. Its job is to look for new services and every time a new service is created it creates the appropriate rules on each node to forward traffic to those services to the backend pods. One way it does this is using IPTABLES rules. In this case it creates an IP tables rule on each node in the cluster to forward traffic heading to the IP of the service which is 10.96.0.12 to the IP of the actual pod which is 10.32.0.15. So how kube-proxy configure the service We discuss a lot more about networking and services kube-proxy and POD networking. Later in this course again we have a large section just for networking. This is a high level overview for now. We will now see how to install kube-proxy. Download the kube-proxy binary from the kubernetes release page. Extract it and run it as a service. The kubeadm tool deploys kube-proxy as PODs on each node. In fact it is deployed as a daemon set, so a single POD is always deployed on each node in the cluster. Well if you don't know about daemon set yet don't worry we have a lecture on that coming up in this course. We have now covered a high-level overview of the various components in the kubernetes control plane.


##POD

kubectl run nginx --image nginx
kubectl get pods


pod-definition.yml

apiVersion: v1
kind: Pod
metedata: 
  name: myapp-pod
  labels:
      app: myapp
      type: front-end

spec:
  containers
     - name: nginx-container
       image: nginx


kubectl create -f pod-definition.yml

kubectl get pods
kubectl describe pod myapp-pod


###KIND       VERSION

###Pod          V1
###Service      V1
###ReplicaSet APPS/V1
###Deployment APPS/V1



kubectl run nginx --image=nginx
kubectl get pods
kubectl describe pod name-of-pod | grep -i image
kubectl describe -o wide

## Create yaml file from kubectl command

kubectl run redis --image=redis123 --dry-run=client -o yaml > pod.yaml


## Replication controller & replica set

Both have the same purpose but they're not the same replication controller is the older technology that is being replaced by replicate set properly has said is the new recommended way to setup replication.
  
  selector is one of the major differences between replication controller and replica set the selector is not a required field in case of a replication controller 
  It assumes it to be the same as the labels provided in the part definition file in case of replica set

apiVersion: v1
kind: ReplicationController
metadata:
  name:
    app: myapp
    type: front-end
spec:
 template:
   metadata:
     name: myapp-pod
     labels:
       app: myapp
       type: frontent
   spec:
     containers:
     - name: nginx-container
       image: nginx
 replicas: 3
 
 
 kubectl create -f rc-definition.yml
 kubectl get replicationcontroller
 
 
 
## replicaset
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
      app: myapp
      type: front-end
spec:
  template:
     metadata:
       name: myapp-pod
       labels:
           app: myapp
           type: front-end
     spec:
       containers:
       - name: nginx-container
         image: nginx
replicas: 3
selector:
   matchLabels:
      type: front-end
   

kubectl get replicaset

increase the number of replicas 
method 1
change replicas:3 to replicas: 6 and run the below command

kubectl replace -f replicaset-definition.yml

method 2
kubectl scale --replicas=6 -f replicaset-definition.yml

kubectl scale --replicas=6 replicaset myapp-replicaset


kubectl delete replicaset myapp-replicaset


apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
        
        
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-2
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
        
        
## Deploymnet

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
      app: myapp
      type: front-end
spec:
  template:
     metadata:
       name: myapp-pod
       labels:
          app: myapp
          type: front-end
     spec:
       containers:
       - name: nginx-container
         image: nginx
  replicas: 3
  selector:
    matchLabels:
       type: front-end
       
 kubectl create -f deployment-definition.yml
 kubectl get deployments
 kubectl get replicaset
 kubectl get pods
kubectl get all



##Namespace

default, kube-system, kube-public 

Default namespace: will be created when cluster created and used for default resource creation 
kube-system namespace: kubernetes will use this for DNS and network pods and service creation 
kube-public namespace: this is where resources that should be made available to all users are created


* Each of the namespaces can have its own set of policies that define who can do what
* You can also assign quota of resources to each of these namespaces that way each name spaces is guranteed a certain amount and does not use more that it's allowed.


Resources in the same namespace can reach the db service simply using the hostname for example

mysql.connect("db-service")

if pod from one namespace what to connect service in another namespace that can be like following

mysql.connect("db-service.dev.svc.cluster.local")

db-service--> service name
dev--> namespace
svc--> service
cluster.local--> domain


kubectl get pods --> which will only list the pods in default namespace
kubectl get pods --namespace=kube-system --> which will list the pods in another namespace

kubectl create -f pod-definition.yml --namespace=dev --> to create pod in particular namespace

or 

apiVersion: v1
kind: Pod
metedata: 
  name: myapp-pod
  namespace: dev
  labels:
      app: myapp
      type: front-end

spec:
  containers
     - name: nginx-container
       image: nginx
       
       
 
 
To create namespace

apiVerion: v1
kind: Namespace
metadata:
  name: dev
  
  
  or
  
kubectl create namespace dev


##To change the namespace context 

kubectl config set-context $(kubectl config currnet context) --namespace=dev
kubectl confit set-context $(kubectl config current context) --namespace=prod

kubectl get pods --all-namespaces


##Resource Quots

apiVersion: v1
kind: ResourceQuota
metadata: 
    name: conpute-quota
    namespace: dev
    
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
    
    
Certification Tip!
Here's a tip!



As you might have seen already, it is a bit difficult to create and edit YAML files. Especially in the CLI. During the exam, you might find it difficult to copy and paste YAML files from browser to terminal. Using the kubectl run command can help in generating a YAML template. And sometimes, you can even get away with just the kubectl run command without having to create a YAML file at all. For example, if you were asked to create a pod or deployment with specific name and image you can simply run the kubectl run command.

Use the below set of commands and try the previous practice tests again, but this time try to use the below commands instead of YAML files. Try to use these as much as you can going forward in all exercises

Reference (Bookmark this page for exam. It will be very handy):

https://kubernetes.io/docs/reference/kubectl/conventions/



Create an NGINX Pod

kubectl run --generator=run-pod/v1 nginx --image=nginx  ( generator deprecated )



Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml
kubectl run nginx --image=nginx --dry-run=client -o yaml


Create a deployment

kubectl create deployment --image=nginx nginx



Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

kubectl create deployment --image=nginx nginx --dry-run -o yaml



Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)

kubectl create deployment --image=nginx nginx --dry-run -o yaml > nginx-deployment.yaml

Save it to a file, make necessary changes to the file (for example, adding more replicas) and then create the deployment.





##sevices


nodeport
clusterip
loadbalancer


kubectl create -f service-definition.yml
kubectl get services

Nodeport uses "Algorighm: Random" "SessionAffinity: Yes"
