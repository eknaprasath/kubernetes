##POD ASSESSMENT

1.how many pods running 

kubectl get pods

2.Create a new pod with the NGINX image

kubectl run nginx --image=nginx

3.What is the image used to create the new pods? You must look at one of the new pods in detail to figure this out

Run the command 'kubectl describe pod newpod-' and look under the containers section.

4. Which nodes are these pods placed on? You must look at all the pods in detail to figure this out

Run the command 'kubectl describe pod newpod-' or 'kubectl get pods -o wide' and look under the containers section.

5. How many containers are part of the pod 'webapp'?

Run the command 'kubectl describe pod webapp' and look under the Containers section.

6. What images are used in the new 'webapp' pod?

Run the command 'kubectl describe pod webapp' and look under the Containers section.

7.What is the state of the container 'agentx' in the pod 'webapp'?

Run the command 'kubectl describe pod webapp' and look under the Containers section.

8. What does the READY column in the output of the 'kubectl get pods'

running containers in a pod and total containers in a pod

9.Delete the 'webapp' Pod.

Run the command 'kubectl delete pod webapp'.

10. Create a new pod with the name 'redis' and with the image 'redis123'. Use a pod-definition YAML file. And yes the image name is wrong!

Create a pod definition YAML file and use it to create a POD or use the command kubectl run redis --image=redis123.

11. Now fix the image on the pod to 'redis'. Update the pod-definition file and use 'kubectl apply' command or use 'kubectl edit pod redis' command.



##REPLICASET

1. How many ReplicaSets exist on the system? in the current(default) namespace

Run the command 'kubectl get replicaset' and count the number of replicasets.

2. How many PODs are DESIRED in the new-replica-set?

Run the command 'kubectl get replicaset' and look at the count under the 'Desired' column

3. What is the image used to create the pods in the new-replica-set?

Run the command 'kubectl describe replicaset' and look under the containers section.

4. How many PODs are READY in the new-replica-set?

Run the command 'kubectl get replicaset' and look at the count under the 'Ready' column

5. Delete any one of the 4 PODs

Run the command 'kubectl delete pod '

6. Why are there still 4 PODs, even after you deleted one?

ReplicaSet ensures that desired number of PODs always run

7. Delete the two newly created ReplicaSets 

Run the command 'kubectl delete replicaset '

8. Fix the original replica set 'new-replica-set' to use the correct 'busybox' image 
Either delete and re-create the ReplicaSet or Update the existing ReplicSet and then delete all PODs, so new ones with the correct image will be created.

Run the command 'kubectl edit replicaset new-replica-set', modify the image name and then save the file.

9. Scale the ReplicaSet to 5 PODs. 
Use 'kubectl scale' command or edit the replicaset using 'kubectl edit replicaset'


Run the command 'kubectl edit replicaset new-replica-set', modify the replicas and then save the file.

10. Now scale the ReplicaSet down to 2 PODs. Use 'kubectl scale' command or edit the replicaset using 'kubectl edit replicaset'

Run the command 'kubectl edit replicaset new-replica-set', modify the replicas and then save the file.













