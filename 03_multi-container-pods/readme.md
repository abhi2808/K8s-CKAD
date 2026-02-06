# multi container pods
when need 2 instances of 2 diff services to pair together but developed and deployed seperately!!

same: lifecycle, network, storage

diff design patterns:
- co-located: run together(cant define order of startup)
syntax:
just add containers to the spec/containers

- regular init: init container runs initialization steps then closes
syntax:
spec:
  containers:....
  initContainers:(can be multiple)
    - name:
      image:
      command:
    - name:
      image:
      command:

- sidecar: sidecar container runs initialization but continues throughout lifecycle
syntax:
spec:
  containers:....
  initContainers:(can be multiple)
    - name:
      image:
      command:
      restartPolicy: Always


in multicontainer if any container fail the pod restarts