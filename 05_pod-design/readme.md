# labels, selectors and annotations
labels(metadata) and selectors help group things/objects together(help filtering)

use: get statement, pod discovery in replicaSet and deployment and servces

annotations store the metadata


# rollout and versioning
when we create a deployment->creates new rollout->creates new replica set(deployment revision)(same flow when we update/modify the deployments this gives us the ability to roll back)

kubectl rollout undo deployment dep-name

kubectl rollout status deployment dep-name

kubectl rollout history deployment dep-name

kubectl rollout history deployment dep-name --revision=3(to get a specific old rollout)

if modifying image using command add: --record to get the cause of change in the annotations

2 general ways to update:
- recreate: causes downtime
- rolling(default): no downtime


other strategies: 
- blue(old)-green(green): used with service meshes, implemented using service labels from old to new
- canary: small % to new, rest to old, slowly  to the new, 2 deployments same labels, %traffic managed using the number of pods in deployments, work better with service meshes can fine-grain


# jobs
a type of workload similar to web, db etc, used for batch processing(short-lived, perform and finish)

apiVersion: batch/v1
kind: Job
metadata:
  name: job-1
spec:
  completions: 3  # pods created one after the other 
  parallelism: 3 # added so that pods can be run parallely
  template:
    spec:
      containers:
        - name: add
          image: ubuntu
          command: ['expr','3','+','2']
      restartPolicy: Never    


# cronJobs
job that can be scheduled 

apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cjob-1
spec:
  schedule: "*/1 * * * *"
  JobTemplate:
    completions: 3   
    parallelism: 3 
    template:
        spec:
          containers:
            - name: add
            image: ubuntu
            command: ['expr','3','+','2']
          restartPolicy: Never  