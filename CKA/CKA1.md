IMPERATIVE VS DECLARATIVE

IMPERATIVE 

kubectl run --image=nginx nginx
kubectl create deployment --image=nginx nginx
kubectl expose deployment nginx --port 80
kubectl edit deployment nginx
kubectl scale deployment nginx --replicas=5
kubectl set image deployment nginx nginx=nginx:1.18
kubectl create -f nginx.yaml
kubectl replace -f nginx.yaml
kubectl delete -f nginx.yaml


DECLARATIVE

kubectl apply -f nginx.yaml


TAINT & TOLERATIONS

Teens and toleration are used to set restrictions on what parts can be scheduled.

kubectl taint nodes node-name key=value:taint-effect

taint-effect = NoSchedule | PreferNoSchedule | NoExecute

If they do not tolerate the taint there are three taint effects no schedule which means the pods will not be scheduled on the node which is what we have been discussing.

Prefer No schedule which means the system will try to avoid placing a pods on the node but that is not guaranteed.

And third is no execute. Which means that new parts will not be scheduled on the node and existing parts on the node if any will be evicted if they do not tolerate the taint.

These parts may have been scheduled on Node before the taint was applied to the node.


kubectl taint nodes node1 app=blue:NoSchedule


TOLERATIONS - PODS

  apiVersion:
  kind: Pod
  metadata:
    name: myapp-pod
  spec:
    containers:
    - name: nginx-container
      image: nginx
      
    tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
      
      
  Remember taints and toleration are only meant to restrict nodes from accepting certain pods.


kubectl describe node controlplane | grep Taints:



NODE SELECTOR

Hello and welcome to this lecture. In this lecturewe will talk about nodes selectors in Kubernetes.Let us start with a simple example.You have a three node cluster of which two are smaller nodes with lower hardware resources and one ofthem is a larger node configured with higher resources you have different kinds of workloads runningin your cluster.You would like to dedicate the data processing workloads that require higher horsepower to the largernode as that is the only node that will not run out of resources in case the job demands extra resources.However, in the current default setup, any pods can go to any nodes.So Pod C in this case may very well end up on nodes two or three which is not desired. To solve thiswe can set a limitation on the pods so that they only run on particular nodes.There are two ways to do this.The first is using Node selectors which is the simple and easier method.For this we look at the pod definition file we created earlier this file has a simple definition to createa pod with a data processing image to limit this pod to run on the larger nodeWe add a new property called Node selector to the spec section and specify the size as large.But wait a minute.Where did the size large come from and how does Kubernetes know which is the large node. The key valuepair of size and large are in fact labels assigned to the nodes the scheduler uses these labels to matchand identify the right note to place the pods on labels and selectors are a topic we have seen many  times throughout this Kubernetes course such as with services replica sets and deployments to use  labels in a known selector like this.  You must have first labelled your nodes prior to creating this pod.  So let us go back and see how we can label the nodes to label a node.  Use the command Kube control label nodes followed by the name of the node and the label in a key value pair format.In this case it would be Kube control label nodes Node 1 followed by the label in a key value formatsuch as size equals large.Now that we have labeled the node we can get back to creating the pod this time with the node selectorset to a size of large.When the pod is now created it is placed on Node 1 as desired not selectors served our purpose but ithas limitations.We used a single label and selector to achieve our goal here.But what if our requirement is much more complex.For example, we would like to say something like place the pod on a large or medium node or somethinglike place the pod on any nodes that are not small.You cannot achieve this using Node selectors for this node affinity and anti affinity features wereintroduced and we will look at that next.

apiVersion:
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  nodeSelector:
    size: Large
    
    
  
 LABEL NODES:
 
   kubectl label nodes <node-name> <label-key>=<label-value>
  
   kubectl label nodes node-1 size=Large
   
   
   
NODE AFFINITY:

  We will talk about known affinity feature in Kubernetes.The primary purpose of node affinity feature is to ensure that pods are hosted on particular notes.In this case to ensure the large data processing pod ends up on no one in the previous lecture we didthis easily using notes selectors we discussed that you cannot provide advanced expressions like oror not with node selectors the node affinity feature provides us with advanced capabilities to limitpod placement on specific nodes with great power comes great complexity.So the simple node selector specification will now look like this with node affinity although both doesexactly the same thing plays the pod on the large node let us look at it a bit closer under spec youhave affinity and then node affinity under that and then you have a property that looks like a sentencecalled required during scheduling ignored during execution no description needed for that and then youhave the node selector terms that is an array and that is where you will specify the key and value pairsthe key value pairs are in the form key operator and value where the operator is in the operator ensuresthat the pod will be placed on a node whose label size has any value in the list of values specifiedhere in this case it is just one called large.If you think your part could be placed on a large or a medium node you could simply add the value tothe list of values like this you could use the not in operator to say something like size not in smallwhere node affinity will match the node with a size not set to small.We know that we have only set the labels size too large and medium nodes the smaller nodes don't evenhave the labels set.So we don't really have to even check the value of the label as long as we are sure we don't set a labelsize to the smaller nodes using the exists operator will give us the same result.The exists the operator will simply check if the label size exists on the notes and you don't need thevalues section for that as it does not compare the values.There are a number of other operators as well.Check the documentation for specific details.Now we understand all of this and we're comfortable with creating a pod with specific affinity ruleswhen the pods are created.These rules are considered and the pods are placed onto the right nodes.But what if node affinity could not match a node with a given expression.In this case what if there are no nodes with a label called size say we had the labels and the podsare scheduled.What if someone changes the label on the node at a future point in time.Will the pod continue to stay on the Node.All of this is answered by the long sentence like property under node affinity which happens to be thetype of node affinity the type of node affinity defines the behaviour of the scheduler with respect tonode affinity and the stages in the lifecycle of the pod.There are currently two types of node affinity available required during scheduling ignored during executionand preferred during scheduling ignored during execution and there are additional types of node affinityplaned.As of this recording required during scheduling required during execution we will now break this downto understand further.We will start by looking at the two available affinity types.There are two states in the lifecycle of a pod when considering node affinity during scheduling andduring execution during scheduling.Is the state where a pod does not exist and is created for the first time.We have no doubt that when a pod is first created the affinity rules specified are considered to placethe pods on the right note.Now what if the nodes with matching labels are not available.For example we forgot to label the node as large.That is where the type of node affinity used comes into play.If you select the required type which is the first one the scheduler will mandate that the pod be placedon a node with a given affinity.Rules if it cannot find one the pod will not be scheduled.This type will be used in cases where the placement of the pod is crucial.If a matching node does not exist the pod will not be scheduled.But let's say the pod placement is less important than running the workload itself.In that case you could set it to preferred and in cases where a matching node is not found.The scheduler will simply ignore node affinity rules and place the card on any available note.This is a way of telling the scheduler hey try your best to place the pod on matching node but if youreally cannot find one just plays it anywhere.The second part of the property or the other state is during execution during execution is the statewhere a part has been running and a change is made in the environment that affects node affinity suchas a change in the label of a node.For example say an administrator removed the label we said earlier called size equals large from thenode.Now what would happen to the pods that are running on the Node.As you can see the two types of node affinity available today has this value set too ignored which meanspods will continue to run and any changes in node affinity will not impact them once they are scheduled.The new types expected in the future only have a difference in the during execution phase a new optioncalled required during execution is introduced which will evict any pods that are running on notesthat do not meet affinity rules.In the earlier example, a pod running on the large node will be evicted or terminated if the label largeis removed from the node.Well that's it for this lecture.Head over to the coding exercises and practice working with node affinity rules in the next lecture.We will compare tables and toleration and node affinity.
  
  
  
  apiVersion:
  kind:
  
  metadata:
    name: myapp-pod
  spec:
    containers:
     - name: data-processor
       image: data-processor
       
    affinity:
      nodeAffinity:
        requireDuringSchedulingIgnoreDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: size
              operator: In
              values: 
              - Large
              
              
              
 Above affinity will place pods on Large nodes if we want to use both Large and Medium
 need to use the below
 
         values:
         - Large
         - Medium
         
We can also use "NotIn" operator as follows

      - key: size
        operator: NotIn
        values:
        - Small
        
        
if we want to place pod if the lable "key" exists then we need to use as folows

     - key: size
       operator: Exists
       
       
       
 NODE AFFINITY TYPES:
 
   Available:
   
   requireDuringSchedulingIgnoredDuringExecution
   preferredDuringSchedulingIgnoredDuringExecution
   
   Planned:
   
   requiredDuringSchedulingRequiredDuringExecution
   
   
   in the above "DuringScheduling" means pods not exising which is going to be created first time
  
             DuringScheduling                 DuringExecution
             
   Type 1     Required( labels mandate  )     Ignored( after scheduling if label delted still pod will rung )
   
   Type 2   Preferred ( if no lables will     Ignored( after scheduling if label delted still pod will rung )
            be placed in any of the node)
  
   Type 3     Required                        Required ( pod will be evicted )
   
   
  There are two states in the lifecycle of a pod when considering node affinity during scheduling andduring execution during scheduling. Is the state where a pod does not exist and is created for the first time. We have no doubt that when a pod is first created the affinity rules specified are considered to place the pods on the right note. Now what if the nodes with matching labels are not available. For example we forgot to label the node as large. That is where the type of node affinity used comes into play. If you select the required type which is the first one the scheduler will mandate that the pod be placed on a node with a given affinity. Rules if it cannot find one the pod will not be scheduled. This type will be used in cases where the placement of the pod is crucial. If a matching node does not exist the pod will not be scheduled. But let's say the pod placement is less important than running the workload itself. In that case you could set it to preferred and in cases where a matching node is not found. The scheduler will simply ignore node affinity rules and place the card on any available note. This is a way of telling the scheduler hey try your best to place the pod on matching node but if you really cannot find one just plays it anywhere. The second part of the property or the other state is during execution during execution is the state where a part has been running and a change is made in the environment that affects node affinity such as a change in the label of a node. For example say an administrator removed the label we said earlier called size equals large from the node. Now what would happen to the pods that are running on the Node. As you can see the two types of node affinity available today has this value set too ignored which means pods will continue to run and any changes in node affinity will not impact them once they are scheduled. The new types expected in the future only have a difference in the during execution phase a new option called required during execution is introduced which will evict any pods that are running on notes that do not meet affinity rules. In the earlier example, a pod running on the large node will be evicted or terminated if the label large is removed from the node.




RESOURCE REQUIREMENTS AND LIMITS

RESOURCE REQUESTS

   By default kubernetes will allocate 0.5 CPU and 256Mi Memory and max limit is 1 cCPU and 512 Mi
   
   
 apiVersion: v1
 kind: Pod
 metadata:
   name: simple-webapp-color
   labels:
     name: simple-webapp-color
     
 spec:
   containers:
   - name: simple-webapp-color
     image: simple-webapp-color
     ports:
       - containerPort: 8080
     resources:
       requests:
         memory: "1Gi"
         cpu: 1
       limits:
          memory: "2Gi"
          cpu: 2
          
          
          
    If a pod trying to use more CPU it will be throttled by the limit but it is not the same case with memory when a 
    pod trying to use more memory than the limit continuously it will be terminated
    
    
When a pod is created the containers are assigned a default CPU request of .5 and memory of 256Mi". For the POD to pick up those defaults you must have first set those as default values for request and limit by creating a LimitRange in that namespace.



apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/



apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 0.5
    type: Container
https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/



References:

https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource



DAEMON SETS

we look at daemon Sets in Kubernetes. So far we have deployed various pods on different nodes in our cluster. With the help of replica sets and deployments we made sure multiple copies of our applications applications are made available across various different worker nodes. Daemon set are like replica sets, as in it helps you deploy multiple instances of pod. But it runs one copy of your pod on each node in your cluster. Whenever a new node is added to the cluster a replica of the pod is automatically added to that node and when a node is removed the pod is automatically removed. The demon set ensures that one copy of the pod is always present in all nodes in the cluster. So what are some use cases of daemonsets? Say you would like to deploy a monitoring agent or log collector on each of your nodes in the cluster so you can monitor your cluster better. Demon set is perfect for that as it can deploy your monitoring agent in the form of a pod in all the nodes in your cluster. Then, you don’t have to worry about adding/removing monitoring agents from these nodes when there are changes in your cluster. as the daemon set will take care of that for you. While discussing the kubernetes architecture we learned that one of the worker node components that is required on every node in the cluster is a kube-proxy. That is one good use case of daemon Sets. The kubeproxy component can be deployed as a daemon set in the cluster. Another use case is for networking. Networking solutions like weave net requires an agent to be deployed on each node in the cluster. We will discuss about networking concepts in much more detail later during this course but I just wanted to point it out here for now. Creating a DaemonSet is similar to the ReplicaSet creation process. It has nested pod specification under the template section and selectors to link the daemon set to the PODs. A daemonSet definition file has a similar structure. We start with apiVersion, kind, metadata and spec. The api version is apps/v1. Kind is DaemonSet instead of ReplicaSet. We will set the name to monitoring daemon. Under spec you have a selector and a pod specification template. It's almost exactly like ReplicaSet definition except that the kind is a demon set. Ensure the labels in the selector matches the ones in the pod template. Once ready create the daemonset using the kubectl create daemonset command. To view the created daemonset run the kubectl get daemonset command. And of course to view more details run the kubectl describe demonSet command. So how does a demon set work. How does it schedule pods on each node? and How does it ensure that every node has a pod? if you were asked to schedule a pod on each node in the cluster. how would you do it? In one of the previous lectures in this section. we discussed that we could set the nodeName property on the pod to bypass the scheduler and get the pod placed on a node directly. So that’s one approach. On each pod, set the nodeName property in its specification before it is created and when they are created, they automatically land on the respective nodes. So that’s how it used to be until kubernetes version v1.12. From v1.12 onwards the Daemon set uses the default scheduler and node affinity rules that we learned in one of the previous lectures to schedule pods on nodes.


apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
	  app: montoring-agent
  template:
    metadata:
	  labels:
	    app: monitoring-agent
    spec:
	  containers:
	  - name: monitoring-agent
	    image: monitoring-agent



apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
     matchLabels:
	   app: monitoring-agent
	template:
	  metadata:
	    labels:
		  app: monitoring-agent
		
	  spec:
	    containers:
		- name: monitoring-agent
		  image: monitoring-agent


ROLLOUT

we will talk about updates and rollbacks in a deployment before we look at how we upgrade our application. Let's trying to understand rollouts and versioning in a deployment when you first create a deployment it triggers a rollout a new rollout creates a new deployment revision let's call it revision 1 in the future when the application is upgraded meaning when the container version is updated to a new one a new rollout is triggered and a new deployment revision is created named revision 2. This helps us keep track of the changes made to our deployment and enables us to roll back to a previous version of deployment if necessary. You can see the status of your rollout by running the command: kubectl rollout status followed by the name of the deployment to see the revisions and history of rollout. run the command kubectl rollout history followed by the deployment name And this will show you the revisions and history of our deployment there are two types of deployment strategies. Say for example you have five replicas of your web application instance deployed. One way to upgrade these to a newer version is to destroy all of these and then create newer versions of application instances meaning first destroy the five running instances and then deploy five new instances of the new application version. The problem with this as you can imagine is that during the period after the older versions are down and before any newer version is up the application is down and inaccessible to users this strategy is known as the Recreate strategy, And thankfully this is not the default deployment strategy. The second strategy is where we did not destroy all of them at once. Instead we take down the older version and bring up a newer version one by one. This way the application never goes down and the upgrade is seamless. Remember if you do not specify a strategy while creating the deployment it will assume it to be rolling update. In other words rolling update is the default deployment strategy so we talked about upgrades. How exactly do you update your deployment when you say update. It could be different things such as updating your application version by updating the version of docker containers used, updating their labels or updating the number of replicas etc Since we already have a deployment definition file it is easy for us to modify this file once we make the necessary changes. we run the kubectl apply command to apply the changes. A new rollout is triggered and a new revision after deployment is created. But there is another way to do the same thing. You could use the kubectl set image command to update the image of your application. But remember doing it this way will result in the deployment definition file having a different configuration so you must be careful when using the same definition file to make changes in the future the difference between the recreate and rolling update strategies can also be seen when you view the deployments in detail. Run the kubectl describe deployment command to see detailed information regarding the deployments. You will notice when the recreate strategy was used. The events indicate that the old replica set was scaled down to zero first and then the new replica sets scaled up to five. However when the Rolling update strategy was used the old replicas set was scaled down one at a time simultaneously scaling up the new replica set one at a time let's look at how a deployment performs an upgrade under the hood when a new deployment is created say to deploy five replicas. it first creates a Replicaset automatically, which in turn creates the number of PODs required to meet the number of replicas. When you upgrade your application as we saw in the previous slide, the kubernetes deployment object creates a new replica set under the hood and starts deploying the containers there at the same time taking down the parts in the old replica set following a rolling update strategy. This can be seen when you try to list the replicasets using the kubectl get replicasets command. Here we see the old replicaset with 0 PODs and the new replicaset with 5 PODs. Say for instance once you upgrade your application you realize something is inferior right. Something's wrong with the new version of build. You used to upgrade so you would like to roll back your update kubernetes deployments allow you to roll back to a previous revision to undo a change. run the command kubectl rollout undo followed by the name of the deployment. The deployment will then destroy the PODs in the new replicaset and bring the older ones up in the old replicaset. And your application is back to its older format. When you compare the output of the kubectl get replicasets command, before and after the rollback, You will be able to notice this difference before the rollback. the first replicaset had 0 PODs and the new replicaset had 5 PODs and this is reversed after the rollback is finished. And finally let’s get back to one of the commands we ran initially when we learned about PODs for the first time. We used the kubectl run command to create a pod. This command in fact creates a deployment and not just a pod. This is why the output of the command says Deployment nginx created. This is another way of creating a deployment by only specifying the image name and not using a definition file the required replica set and parts are automatically created in the back end using a definition file is recommended though as you can save the file check it into the code repository and modify it later as required. To summarize the commands real quick, use the kubectl create command to create the deployment, get deployment command to list the deployments apply and set image commands to update the deployments and rollout status command to see the status of rollouts and rollout undo command to roll back a deployment. Operation.

  kubectl rollout status deployment/myapp-deployment
  
  kubectl rollout history deployment/myapp-deployment
  
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: myapp-deployment
    labels:
        app: myapp
	type: front-end
  spec:
    template:
       matadata: myapp-pod
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
	  
	  
	  
modify the file  and do the deployment multiple times

kubectl apply -f deployment.yaml
  
  
  
 StrategyType:  Recreate  &  RollingUpdate
 
 Recreate will bring down old deployment to 0 and it will create new deployment
 
 Rolling update will bring one pod down and it will create new pod 
 
 
 
 kubectl rollout undo deployment/myapp-deployment
 
 
 
 ROLLOUT COMMANDS
 
  Create   kubectl create -f deployment.yaml
  
  Get      kubectl get deployments
  Update   kubectl apply -f deployment.yaml
           kubectl set image deployment/myapp nginx=nginx:1.9.1
  status   kubectl rollout status deployment/myapp
           kubectl rollout history deployment/myapp
  Rollback kubectl rollout undo deployment/myapp
  
  
  
  
  COMMANDS AND ARGUMENTS
  
  apiVersion: v1
  kind: Pod
  metadata: 
    name: ubuntu-sleeper-pod
  spec:
    containers:
      - name: ubuntu-sleeper
        image: ubuntu-sleeper
	
  
  ENTRYPOINT ["sleep"]  ==  command: ["sleep2.0"]
  entrypoint in docker is command in pod definition
  
  
  CMD ["5"] == args: ["10"]
  cmd in docker is args in pod definition
  
  
  
  ENVIRONMENT VARIABLES
  
  docker run -e APP_COLOR=pink simple-webapp-color
  
  apiVersion: v1
  kind: Pod
  metadata:
    name: simple-webapp-color
  spec:
    containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      env:
        - name: APP_COLOR
	  value: pink
  
  
  
  ENV VALUE TYPES
  
  PLAIN KEY VALUE    -->  env:
                            - name: APP_COLOR
			      value: pink
			      
  CONFIGMAP          -->  env:
                            - name: APP_COLOR
			      valueFrom:
			          configMapKeyRef:
				  
  SECRETS            -->  env:
                            - name: APP_COLOR
			      valueFrom:
			          secretKeyRef:
				  
				  
				  
				  
ConfiMaps

Imperative:  kubectl create configmap

             kubectl create configmap <config-name>  --from-literal=<key>=<value>
	     
	     kubectl create configmap app-config --from-literal=APP_COLOR=blue
	     
	     with multiple values
	     type 1
	     kubectl create configmap app-config --from-literal=APP_COLOR=blue  \
	                                         --from-literal=APP_MOD=prod
						 
	     type 2
	      with config file
	      kubectl create configmap <config-name> --from-file=<path-to-file>
	      
	      kuebctl create configmap app-config --from-file=app_config.properties

Declarative: kubectl create -f

           apiVersion: v1
	   kind: ConfigMap
	   metadata:
	      name: app-config
	   data:
	      APP_COLOR: blue
	      APP_MODE: prod
	      
	      
	      
	   kubectl get configmaps
	   
	   kubectl describe configmaps
	   
	   
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
   containers:
   - name: simple-webapp-color
     image: simple-webapp-color
     ports:
       - containerPort: 8080
     envFrom:
       - configMapRef:
             name: app-config
	     
	     
SINGLE ENV 

    env:
      - name: APP_COLOR
        valueFrom:
	   configMapKeyRef:
	      name: app-config
	      key: APP_COLOR
	      
	      
VOLUME

  volumes:
  - name: app-config-volume
    configMap:
       name: app-config
       
       
       
       
       
SECRETS

encrypt
  echo -n 'mysql' | base64
  
decrypt

  echo -n 'bXlzcWW=' | base64 --decode
  
  
    envFrom:
      - secretRef:
            name: app-secret
	    
SINGLE ENV	    
    env:
       - name: DB_PASSWORD
         valueFrom:
	   secretKeyRef:
	      name: app-secret
	      key: DB_Password
	      
	      
VOLUME

volumes:
- name: app-secret-volume
  secret:
     secretName: app-secret
