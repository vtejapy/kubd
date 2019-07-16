# job 
- each job creates one or more pods
- Ensure they are successfully terminated
- job controller restarts or re-scheduled if a pod node fails during execution
-  can run multiple pods in parallel 
-  can scale up using kubectl scale command

***useCases***:
- one time initilization of resource such as databases
-  multiple worker to process message in queue

Jobs.yaml
```
apiVersion: batch/v1
kind: Job
metadata:
  name: countdown
spec:
  template:
    metadata:
      name: countdown
    spec:
      containers:
      - name: counter
        image: centos:7
        command:
         - "bin/bash"
         - "-c"
         - "for i in 9 8 7 6 5 4 3 2 1 ; do echo $i ; done"
      restartPolicy: Never
```
