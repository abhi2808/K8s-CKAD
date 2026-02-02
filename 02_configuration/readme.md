# docker

CMD vs  ENTRYPOINT-> acts as arguments, replaced and uses bash vs acts as actual executable(both run on startup)

FROM Ubuntu
CMD sleep 10
cli-> docker run ubuntu-sleeper sleep 10

FROM Ubuntu
ENRTYPOINT ["sleep"]
cli-> docker run ubuntu-sleeper 10(would error if you dont dive 10)

to preven use both:
FROM Ubuntu
ENRTYPOINT ["sleep"]
CMD ["10"]

the params you pass in cli take precedence

if in k8s,
after name, image pass:
args: ["10"] (overrides the cmd)
command: ["sleep2.0"] (overrides the entrypoint)


once created other than the spec.containers[*].image, spec.initContainers[*].image, spec.activeDeadlineSeconds, spec.tolerations no specifications can be edited, if needed you can:

- edit the pod(creates a temp config file)->delete the curr pod->create new uising the tmp file

- export the yml->modify->delete->create


for deployment:
- edit field property->would delete+recreate pods according to new config


# using env variables

- key-value
env:
  - name: app_color
    value: pink

- config map
env:
  - name: app_color
    valueFrom:
      configMapKeyRef:
        name: (of the configmap)
        key: (the key to reference)
 

- secrets
env:
  - name: app_color
    valueFrom:
      secretKeyRef:
        name: (of the configmap)
        key: (the key to reference)


OR

envFrom:
  - configMapRef/secretRef:
      name: (name)


# config maps
pass config data in key-value pair as text form

imperitive:
kubectl create configmap <map-name> --from-literal=color(key)=blue(value)

kubectl create configmap <map-name> --from-file=(path)

declarative:
apiVersion: v1
kind: ConfigMap
metadata: 
  name: app-config
data: 
  color: blue


# secrets
like configmaps but hashed

imperitive:
kubectl create secret generic <secret-name> --from-literal=color(key)=blue(value)

kubectl create secret generic <secret-name> --from-file=(path)


declarative:
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  key: value(should be base 64 encoded)


echo -n "abhinav" | base64

can be ejected -> indivisually, all, volume

volume:
  - name: app-secret-volume(volume name)
    secret(volume type): 
      secretName: app-secret(secret obj)

(each secfret attribute stored as a file)


# Secret store CSI driver:

base64 => not secure, also secrets stored in places like vault, secret managers etc => central location, thus tool req to syn these external secrets with our cluster.

Secret store CSI driver synchronizes secrets from external API and mouts them to containers as volumes

how diff from eso and sealed secrets: 
- no need for k8s secrets(less attack surface)

generally installed using helm-> helm install

custom resource definitions installed=>

in secret provider class(talks to the concerned store):
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata: 
  name: db-secrets
spec:
  porvider: aws
  parameters:
    objects: |
        - objectName: "db-creds" (secret name from alert manager)
          objectTyps: "secretsmanager"

in pod spec:

spec:
  volumes:
    - name: db-creds
      csi:
        driver: secret-store.csi.k8s.io
        readOnly: true
        volume attributes:
          secretProviderClass: db-secrets
  container:
    - name: nginx
      image: nginx
      volumeMounts:
        - name: db-creds
          mountPath: /tmp


# secret data

how data is stored in etcd: registry/secrets/default/(secret name)

apt-get install etcd-client: a etcd-control plane pod is created in the kube-system namespace

you can use etcdctl to retrive the data, thus encryption at rest is necessary

apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers: (the order of the provider matters, the top one is used to encrypt)
      - identity: {} (=>no encryption)
      - aesgcm:
          keys:
            - name: key1
              secret:12345678
      - aescbc
          keys:
            - name: key1
              secret:12345678
      - secretbox:
          keys:
            - name: key1
              secret:12345678

to use this file, add in the command section on the container in the kube-apiserver(etc/kubernetes/manifests/kube-apiserver.yml) pod config: - encryption-provider-config/etc/kubernetes/enc/enc.yml(file name). Only new data in encrypted


# docker security
as host and container have the same kernel they are seperated by only namespace. all processes on a container run on a diff namespaces, these process may have diff pid in diff namespaces.(<-process isolation)

by default processes run on root user, if diff can define using --user in the docker run command, or in the dockerfile metion USER 1000(say)

root user container(only for the capabilities docker has, to add priviledges use: --cap-add (per-name), all priviledges: --privileged) not equal to root user on host

for in kubernetes, in pod or container add the following in the spec ot containers section:

securityContext:
  runAsUser: 1000
  capabilities:
    add: ["MAC_ADMIN"]


# resource requirements

kube-scheduler assigns pods to nodes based on available and required resources, if not enough resources->pod in pending stage(+shows insufficeient resources)

you can set the cpu and mem req by a container -> resource request

spec/in containers:
  resources:
    requests:
      memory: "4Gi"(256Mi)
      cpu: 2(1=1 vCPU)
    limits:
      memory: "6Gi"
      cpu: 3

1G= gigabyte, 10^9bytes
1Gi= gibibyte, 1024 Mi

a container can take as many resources as it wants on a node. May suffocate others

CPU cant exceed the limit, memory can(if uses more than limit continuously the pod is terminated with OOM(out of memory) errors)

BEST CPU: set requests, no limits
BEST Memory: all 4 combinations

what can we do:
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
    - default:(limit)
        cpu: 500m
      defaultRequest:(request)
        cpu: 500m
      max:(limit)
        cpu: "1"
      min:(request)
        cpu: 100m
      type: ontainer


if set boundries at "node level":
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    requests.cpu: 4
    requests.memory: 2Gi
    limits.cpu: 5
    limits.memory: 5Gi


# service accounts

linked to authentication, authorization, RBAC(but they are not in CKAD)

accounts:
- user: used by humans
- service: used by machines eg prometheus

kubectl create serviceaccount dashboard-sa, also create sa token automatically, this is used by external apps to authentcate to k8s api, but this token is stored as a secret object(acts as the authentication bearer token for REST class)

old={

if 3rd party app on k8s itelf, instead of copying mount the token secret as volume inside the pod of the 3rd party app.

a default SA(very restricted) is created automatically + token mounted automatically too for the default service account

to add->
spec: 
  serviceAccount: ...

SA of a pod can't be changed ,pod needs to be recreated to change, if you dont want auto attaching of the service AC->
spec:
  automountServiceAccountToken: false

}

now 1.22, TokenRequestAPI: 
no reliance on the SA token, instead a token(defined lifetime) is generated using SA admission controller ad mounted as a projected volume to the pod

1.24, creating SA doen't create a secret: kubectl create token dashboard-sa(SA name), if now want the old like token(no lifetime)(not preffered though):

apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  annotations:
    kubernetes.io/service-account.name: dashboard-sa


# taints and tolerations
restrict what pods can be scheduled on what node. generally pods are generally distributed evenly on the k8s nodes by the k8s schedular

if specific pods on a node:

1) taint the node, by default pods dont have tolerations so cant handle any taint so none can be placed

2) add that toleration to the pod you want on that node

taint: node, tolerations: pod


kubectl taint nodes node-name key=value:taint-effect

taint-effect: what would happen to the pod if if doesn't tolerate the taint:

- NoSchedule: wont be scheduled on that node
- PreferNoSchedule: prefer not to but no guarantee
- NoExecute: new not scheduled, old evicted(kill) if dont tolerate

syntax:
spec:
  tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"

doesnot tell pod to go to a particular node, tells node to only accept certain pods 

Note: This is How master node avoids scheduling any pods. 

untaint: kubectl taint nodes node-name key:effect-


# node selectors

3 nodes-> 1 big, 2 small, but curr any load can go to any pod.

in pod,
spec:
  nodeSelector:
    size: Large

node should be labeled before this: kubectl label node node-name key=value

if you want better control like place pod on large or medium or, not on small node then u need node affinity and anti affinity


# node affiity
ensure pods are hosted on perticular nodes(advanced pod placement), eg to replicate previous funtionality

in pod def,
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: In/NotIn
                values: 
                  - Large
                  - Medium 

what if the node doesn't have the label, or pod is labeled at a later stage, that is handeled by affinity types:

1) requiredDuringSchedulingIgnoredDuringExecution: if node not labeled, pod not scheduled
2) prefferedDuringSchedulingIgnoredDuringExecution: if no matching then too place


Note: Thus to finegrain the placement of pods into the required nodes a combination of tainnts-tolerations and node affinity is used!! 