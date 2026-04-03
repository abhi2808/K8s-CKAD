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
    - Egress
  ingress:            # here elements of array mean "or", while if in array element = map that means "and"
    - from: 
        - podSelector:
            matchLabels:
              name: api-pod
        - nameSpaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: prod
        - ipBlock:
            cidr: 192.168.5.10/32
      ports:
        - protocol: TCP
          port: 3306 
  egress:
    - to:
        - ipBlock:
            cidr: 198.168.5.10/32
      ports:
        - protocol: TCP
          port: 80

solutions that support network policies: kube-router, calico, romana

doesn't: flannel


# Ingress
allows users to acess your app as a single externally accessible url+SSL

1) deploy ingress controller, nginx, ha proxy, istio etc. Deploy using a deployment file with args in image set tp path and pointing to config to store configs+ports to open, 2 env vars of for pod and namespace to deploy to.


2) create ingress resources(set of rules)

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
    - http:
        paths:
          - path: /wear
            pathType: Prefix
            backend:
              service:
                name: wear-service
                port: 
                  number: 80
          - path: /watch
            pathType: Prefix
            backend:
              service:
                name: watch-service
                port: 
                  number: 80


apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
    - host: www.wear.com
      http:
        paths:
          - path: /wear
            pathType: Prefix
            backend:
              service:
                name: wear-service
                port: 
                  number: 80
    - host: www.watch.com
      http: 
        paths:
          - path: /watch
            pathType: Prefix
            backend:
              service:
                name: watch-service
                port: 
                  number: 80


to add tls certificates:

annotations:
.............ssl-redirect:......

spec:
  tls:
    - hosts:
        -........
      secretName: tls-secret

nginx.ingress.kubernetes.io/rewrite-target changes the request URL path before forwarding it to the backend service. It lets Ingress expose apps under paths like /watch or /wear while the backend application still receives requests at /. eg ingress/path => service/