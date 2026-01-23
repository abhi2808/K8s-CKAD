# namespaces

created at cluster creation:
- default namespaces
- kube-system: networking+dns
- kube-public: resources for all users

use: say same cluster for prod and dev, but wnat to keep resources seperated(can asssign quota to each namespace) can communicate with services from diff namespaces, just append the namespace to the name of the service. eg to access the service in dev namespace: db-service.dev.svc.cluster.local

when using commands use eg: kubectl get pods --namespace=kube-system

switch namespace: kubectl config set-context $(kubectl config current-context) --namespace=dev

to get from all: kubectl get pods --all-namespaces

creating quota:
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi


# imperitive commands

one time tasks

--dry-run: creates the resource
--dry-run=client: tells if resource can be created
-o yaml: gives resource def yaml 

can use like: 

pods->
kubectl run nginx --image=nginx --labels tier=db --dry-run=client -o yaml>nginx-pod.yaml


Deployment->
kubectl create deployment --image=nginx nginx
Generate Deployment YAML file (-o yaml): kubectl create deployment --image=nginx nginx --dry-run -o yaml
Generate Deployment with 4 Replicas: kubectl create deployment nginx --image=nginx --replicas=4
to scale: kubectl scale deployment nginx --replicas=4


Service->
Service named redis-service of type ClusterIP to expose pod redis on port 6379: 
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
(This will automatically use the pod's labels as selectors, if NodePort: type:NodePort but cannot specify the node port do in def file)

kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml (This will not use the pods' labels as selectors; instead it will assume selectors as app=redis. You cannot pass in selectors as an option.but can specify the nodeport if you are using it --node-port=30080)


for all commands can modify the output format using: 
-o json: JSON formatted API object.
-o name: Print only the resource name.
-o wide: plain-text format with any additional information.
-o yaml: YAML formatted API object.


other useful commands->
- kubectl api-resources: api version for all resources
- kubectl explain pods/pods.spec.container(can filter): lists few properties and sub properties
- kubectl explain pods --recursive: provides all fields

hpa: horizontal pod auto scalar
namespaced: clusterip scoped