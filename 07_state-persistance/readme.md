# Storage in docker
storage drivers and volume drivers

file system: var/lib/docker -> containing aufs, containers, image, volumes etc.

layered architecture: layers reused across images! 

If I create a file in container, created in container layer ie read and write, while the image layer(shared across multiple containers) is read only. Code written lies in the image layer only, when changes need to be made for modifying a copy of the file is created in the read write layer, thus the image doesn't change unless its rebuilt.

the container is deletable thus, how can we make that data persist by attatching a voulme(creates a folder data_volume in the volume folder). This volume is attatched to the containers.

old: docker run -v data_volume:/var/lib/mysql mysql

new: docker run --mount type=bind, source=data/mysql,target=/var/lib/mysql mysql


when you mount data from the set var/lib/volumes folder then volume mount, if data is already existing on some other path then bind.

storage driver: aufs, zfs, overlay, device mapper(deplends on the os)

volumes are handled by volume drivers: local(default), on 3rd party solutions: azure file storage, convaoy, flocker, rezray(aws)


# Volumes in k8s

in pod def:

spec:
  containers:
    - image: ....
      name: abhi
      command: []
      args: []
      volumeMounts:
        - mountPath: /opt
          name: data-volume
  volumes:
    - name: data-volume
  hostPath: 
    path: /data
    type: Directory


say want ebs as volume:

volumes:
    - name: data-volume
      awsElasticBlockStore:
        volumeID: <id>
        fsType: ext4


# Persistent Volumes
A cluster wide pool of storage created by admins to be used as storage by the user by creating claims from it.

apiVersion: v1
kind:
metadata:
  name: pv-vol1
spce:
  accessModes:
    - ReadWriteOnce (or ReadOnlyMany, ReadWriteMany)
  capacity:
    storage: 1Gi
  hostPath: (can replace with ebs etc)
    path: /tmp/data


# Persistent Volume Claim
created by user, after creation bound to the PVs based on request/properties(sufficient capacity), 1 claim bound to 1 volume(even if some part of volume remains unused). If multiple volumes can fit the claim then we ca use labels to select. If no volume available claim will wait till it is.

apiVerison: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes: 
    - ReadWriteOnce(should match with the PV)
  resources:
    requests:
      storage: 500Mi


persistentVolumeReclaimPolicy: Retain(default)/Delete/Recycle, in the PV config!


Once you create a PVC use it in a POD definition file by specifying the PVC Claim name:

apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim


Say you create a PV, PVC and a pod with that PVC attatched! if you try to delete the PVC it isn't instanatly deleted, it gets stuck in Terminating state as it is being used by a pod, but we can check as soon as the pod is deleted the PVC deletion is also completed and the PV can be seen in the released state but it still has the PVC data, what will happen to that data depends on reclaim policy