# Observability


# readiness probe 

condition of a pod tells bout its various stages(initialized, pods scheduled, ready)

sometimes the pod may say its ready but the contents inside it may not actually have been created, this may cause issue as services expose pods/nodes based on their ready state. Thus we need to set up things that actually check if pods are ready(probes: http, tcp test, exec commands)

in pod spec->

1) http->
containers:
  readinessProbe:
    httpGet:
      path: /api/ready
      port: 8080
    initialDelaySeconds: 10 (after what time to start)
    periodSeconds: 5 (at what intervals)
    failureThreshold: 8 (if not up for this many attempts then fail, default: 3)

2) tcp->
containers:
  readinessProbe:
    tcpSocket:
      port: 3360

3) exec->
containers:
  readinessProbe:
    exec: 
     command:
       - cat 
       - /app/is ready


# liveness probe
generally the container goes down-> pod goes down and is recreated but say application not working but container continues to stay alive(say syntax error) container needs to be destroyed and new one needs to be brought up. Liveness probe checks if the application inside a container is healthy(web-app: api running, db: tcp listening), if fail destroy the container and recreate.

syntax: same as readiness probe!!


# application logs
docker logs -f containerid
kubectl logs -f pod-name

if multiple containers:
ubectl logs -f pod-name container-name(else fail)


k8s no native monitoring solution, 3rd party: metrics server(modern version of the depricated heapster), prometheus, elastic stack, datadog, dynatrace

metrics server(done using cadvisor on kubelet)-> retrives metrics form nodes, in-memory(no historic data)