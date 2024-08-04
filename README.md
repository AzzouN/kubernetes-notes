# Kubernetes

## Notes

- A Kubernetes cluster consists of a set of nodes, we have to type of nodes, working node (a node that host the containters (application)) and master node (a node that control and manage the working nodes)
- We use the kube-apiserver to interact with all the components of the master node (ETCD cluster, kube-scheduler, controller manager)
- Every working node have a kubelet which is an agent that listen to kube-apiserver instructions to either control the containers or to report its status
- Every working node havve kube-proxy which is a component that ensures the communication between different services (containers => working nodes)
- **ETCD** is a distributed reliable key-value database
- apiVersion of replicaSet is apps/v1
- Labels of selector in replicaset must match labels of template
- nodePort range: 30000 - 32767
- We have 3 types of services : ClusterIP (default), NodePort, LoadBalancer
- NodePort service exposes the service to external clients (outside the node)
- ClusterIP service allows internal access (inside the node) but we can expose the service using Ingress
- Creation of NodePort => creation of ClusterIP + opening a port on the node
- Creation of LoadBalancer => creation of ClusterIP + opening a port on the node that is accessible only by the load balancer of the cloud
- NodePort has security issues and it is usually used for debugging and never in production (because we open ports on the node for everyone), the alternative is LoadBalancer, it’s exactly like NodePort but the port is accessible only by the load balancer component on the cloud so the entry point becomes the load balancer of the cloud
- To Access a service in other namespace, use a DNS (The DNS line is created automatically) of the following format: {component name}.{namespace}.svc.cluster.local (ex: dev.svc.cluster.local)
- We can access a service directly using its name if we are in the same namespace
- A schedular is a component that decide the node that will host the pod
- The schedular algorithm runs only for the pods that does not have nodeName property
- If we dont have any schedular and the pod is not assigned to a node using nodeName, the pod will stay on Pending state
- We can't directly assign the node to a Pod when it is already created. To do so we create a Binding object and send a POST request to the Pod's binding API
- All the master node components (Schedular, API Server, Controller Manager ...etc) are  pods in the namespace kube-system
- When we specify multiple selectors, the objects that will be selected are the objects that match all selectors
- annotations are key value pairs used to add addition descriptive informations for objects and can't not be used as selectors
- Taint and toleration is a cencept used in scheduling to allow/prohibit a pod to be in a node (taint for nodes, toleration for pods) (By default, the pods dont have any toleration)
- **important: Taint and toleration does not tell the pod to which nodes must go, but it tells the node which pods to accept (of course the schedular do this work)**
- The schedular does not schedul any pods on the master node because at the installation step, they added a taint to the master node
- nodeSelector is a property that we can add to a pod definition file so the schedular will be able to schedule this pod only on the nodes that have the specefied key value pairs mentioned in NodeSelector (limited, check affinify)
- **nodeName** is a property that we can add to a pod definition file in order to assign the pod to the specific node (this is not done via the schedular)
- **affinity** is a property that we add to the pod definit file in order to specify advanced conditions on how to place the pod on nodes.
- We can specify the resources required (minimum) and resources limit (maximum) for a **container** of a pod in the pod definition file (cpu, memory)
- 1 cpu means 1 vCPU, minimum is 1m vCPU which equals to 0.001 vCPU
- memory => xG, xM, xK, xGi, xMi, xKi
- The system throttle the cpu usage when it reachs the specified limit so the container can never exceed the limit, however, if the container exceed the memory limit, it will be terminated (Out Of Memory error)
- By default, the pods have no resources requests nor limits, so this can affect the other processes running on the node
- We can configure a default resources and limits in **namespace level** using **LimitRange** object, and the values are applied on containers creation, so if we edit the LimitRange it wont affect the already running containers
- **ResourceQuota** is an object that sets the resources requests and limits for namespaces so we can control the aggregated resources used by all of the pods of a namespace
- 

## Components

- Pod: the smallest unit in kubernetes, it usually contain one container
- ReplicaSet: a component that spawns and monitor pods
- Deployment: a component that create a ReplicaSet with additional features (update, rollback, pause, resume)
- Service: a component that act like a router or port forwarding (enables accès to pods)
- Endpoint: a component that represent the matched pod to a service (created for every identified pod for a service)
- Namespace: an isolation layer of components (by default, k8s create 3 namespaces: kube-system, kube-public, Default)
- ResourceQuota: a component in which we define resource specifications and then we assign it to a namespace
- Binding: a component that we use in order to change the nodeName for already created pods (we send the object as JSON to the Pod API)

## Commands

Get informations

```bash
kubectl get pods
kubectl describe pod newpods-424xm
kubectl get replicasets
kubectl describe replicaset new-replica-set
kubectl get all
kubectl get service
kubectl describe service kubernetes
kubectl get endpoints
kubectl get pods --namespace=kube-system
kubectl get pods --all-namespaces
k get pods --selector='env=dev'
```

Create objects (Imperative)

```bash
kubectl run nginx --image=nginx
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
kubectl create -f file.yaml --namespace=dev
kubectl create namespace dev
kubectl run redis --image=redis -n finance
kubectl expose pod redis --port=6379 --name=redis-service
kubectl run redis --image=redis:alpine --labels='tier=db'
```

Update objects (Imperative)

```bash
kubectl config set-context $(kubectl config current-context) --namespace=dev
kubectl scale --replicas=4 replicaset new-replica-set
kubectl delete pod webapp
kubectl edit replicaset new-replica-set
kubectl delete pods -l name=busybox-pod
kubectl expose deployment nginx --port 80
kubectl taint nodes node-name key=value:taint-effect
# taint-effect: the action when the pod is not tolerant; 
# taint-effect: NoSchedule | PreferNoSchedule | NoExecute
# NoSchedule : Do not schedule next pods if they do not tolerate
# PreferNoSchedule : Avoid scheduling the non tolerant pods but its not guaranteed
# NoExecute : NoSchedule + stop all the running pods that does not tolerate
kubectl taint nodes node-name key=value:taint-effect-
kubectl label nodes node-name key=value
```

Declarative

```bash
kubectl apply -f nginx.yaml
```
