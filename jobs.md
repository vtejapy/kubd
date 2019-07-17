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

Example: 2

job-prime.yaml

```
apiVersion: batch/v1
kind: Job
metadata:
  name: primes
spec:
  template:
    metadata:
      name: primes
    spec:
      containers:
      - name: primes
        image: ubuntu
        command: ["bash"]
        args: ["-c",  "current=0; max=110; echo 1; echo 2; for((i=3;i<=max;)); do for((j=i-1;j>=2;)); do if [  `expr $i % $j` -ne 0 ] ; then current=1; else current=0; break; fi; j=`expr $j - 1`; done; if [ $current -eq 1 ] ; then echo $i; fi; i=`expr $i + 1`; done"]
      restartPolicy: Never
  backoffLimit: 4
```

Let’s save this spec in the job-prime.yaml  and create the job running the following command:
```
kubectl create -f job-prime.yaml
job.batch "primes" created
```
Next, let’s check the status of the running job:
```
kubectl describe jobs/primes
Name:           primes
Namespace:      default
Selector:       controller-uid=2415e4c1-802d-11e8-a389-0800270c281a
Labels:         controller-uid=2415e4c1-802d-11e8-a389-0800270c281a
                job-name=primes
Annotations:    <none>
Parallelism:    1
Completions:    1
Start Time:     Thu, 05 Jul 2018 11:26:40 +0300
Pods Statuses:  0 Running / 1 Succeeded / 0 Failed
Pod Template:
  Labels:  controller-uid=2415e4c1-802d-11e8-a389-0800270c281a
           job-name=primes
  Containers:
   primes:
    Image:      ubuntu
    Port:       <none>
    Host Port:  <none>
    Command:
      bash
    Args:
      -c
      current=0; max=110; echo 1; echo 2; for((i=3;i<=max;)); do for((j=i-1;j>=2;)); do if [  `expr $i % $j` -ne 0 ] ; then current=1; else current=0; break; fi; j=`expr $j - 1`; done; if [ $current -eq 1 ] ; then echo $i; fi; i=`expr $i + 1`; done
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  46s   job-controller  Created pod: primes-bwdt7
```
Pay attention to several important fields in this description. In particular, the key Parallelism  has a value of 1 (default) indicating that only one pod was started to do this job. In its turn, the key Completions  tells that the job made one successful completion of the task (i.e prime numbers calculation). Since the pod successfully completed this task, the job was completed as well. Let’s verify this by running:
