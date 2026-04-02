# Networking

ingress(inbound), egress(outbound)
by default, k8s has an all allow rule for communication of pods in a cluster

to prevent/modify traffic flow we can create a network policy(another k8s object), to implement we use labels(on pods) and selectors(network policy)

apiVersion: networking.k8s.io.v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels: 
      role: db
  policyTypes:
    - Ingress
  ingress:
    - from: 
        - podSelector:
            matchLabels:
              name: api-pod
      ports:
        - protocol: TCP
          port: 3306 

solutions that support network policies: kube-router, calico, romana

doesn't: flannel