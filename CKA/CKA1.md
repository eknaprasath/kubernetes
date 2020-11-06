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
