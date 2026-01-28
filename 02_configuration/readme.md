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