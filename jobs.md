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
```
kubectl get jobs
NAME      DESIRED   SUCCESSFUL   AGE
primes    1         1            5m
 ```
You can also easily check the prime numbers calculated by the bash script. In the bottom of the job description, find the name of the pod created by the Job (in our case, it is a pod named primes-bwdt7  . It is formatted as [POD_NAME][HASH_VALUE}  ). Let’s check the logs of this pod:
```
kubectl logs primes-bwdt7
1
2
3
5
7
11
13
17
19
23
29
31
37
41
```
 
That’s it! The pod created by our Job has successfully calculated all prime numbers between 0 and 110. Above example represents a non-parallel Job. In this type of Jobs, just one pod is started unless it fails. Also, a Job is completed as soon as the pod completes successfully.

However, the Job controller also supports parallel Jobs which can create several pods working on the same task. There are two types of parallel jobs in Kubernetes: jobs with a fixed completions count and parallel jobs with a work queue. Let’s discuss both of them.

Jobs with a Fixed Completions Count
Jobs with a fixed completions count create one or more pods sequentially and each pod has to complete the work before the next one is started. This type of Job needs to specify a non-zero positive value for .spec.completions  which refers to a number of pods doing a task. A Job is considered completed when there is one successful pod for each value in the range 1 to .spec.completions  (in other words, each pod started should complete the task). The jobs of this type may or may not specify the .spec.parallelism  value. If the field value is not specified, a Job will create 1 pod. Let’s test how this type of jobs works using the same spec as above with some slight modifications:

```
apiVersion: batch/v1
kind: Job
metadata:
  name: primes-parallel
  labels:
     app: primes
spec:
  completions: 3
  template:
    metadata:
      name: primes
      labels:
        app: primes
    spec:
      containers:
      - name: primes
        image: ubuntu
        command: ["bash"]
        args: ["-c",  "current=0; max=110; echo 1; echo 2; for((i=3;i<=max;)); do for((j=i-1;j>=2;)); do if [  `expr $i % $j` -ne 0 ] ; then current=1; else current=0; break; fi; j=`expr $j - 1`; done; if [ $current -eq 1 ] ; then echo $i; fi; i=`expr $i + 1`; done"]
      restartPolicy: Never
```
The only major change we made is adding the .spec.completions  field set to 3, which asks Kubernetes to start 3 pods to perform the same task. Also, we set the app:primes  label for our pods to access them in kubectl  later.

Now, let’s open two terminal windows.

In the first terminal, we are going to watch the pods created:
```
kubectl get pods -l app=primes -w
``` 
Save this spec in job-prime-2.yaml  and create the job running the following command in the second terminal:

```
kubectl create -f job-prime-2.yaml
job.batch "primes-parallel" created
``` 
Next, let’s watch what’s happening in the first terminal window:
```
kubectl get pods -l app=primes -w
NAME                    READY     STATUS    RESTARTS   AGE
primes-parallel-7gsl8   0/1       Pending   0          0s
primes-parallel-7gsl8   0/1       Pending   0         0s
primes-parallel-7gsl8   0/1       ContainerCreating   0         0s
primes-parallel-7gsl8   1/1       Running   0         3s
primes-parallel-7gsl8   0/1       Completed   0         14s
primes-parallel-nsd7k   0/1       Pending   0         0s
primes-parallel-nsd7k   0/1       Pending   0         0s
primes-parallel-nsd7k   0/1       ContainerCreating   0         0s
primes-parallel-nsd7k   1/1       Running   0         4s
primes-parallel-nsd7k   0/1       Completed   0         14s
primes-parallel-ldr7x   0/1       Pending   0         0s
primes-parallel-ldr7x   0/1       Pending   0         0s
primes-parallel-ldr7x   0/1       ContainerCreating   0         0s
primes-parallel-ldr7x   1/1       Running   0         4s
primes-parallel-ldr7x   0/1       Completed   0         14s
```
 
As you see, the kubectl  started three pods sequentially waiting for the current pod to complete the operation before starting the next pod.

For more details, let’s check the status of the Job again:
```
kubectl describe jobs/primes-parallel
Name:           primes-parallel
Namespace:      default
Selector:       controller-uid=2ec4494f-8035-11e8-a389-0800270c281a
Labels:         controller-uid=2ec4494f-8035-11e8-a389-0800270c281a
                job-name=primes-parallel
Annotations:    <none>
Parallelism:    1
Completions:    3
Start Time:     Thu, 05 Jul 2018 12:24:14 +0300
Pods Statuses:  0 Running / 3 Succeeded / 0 Failed
Pod Template:
  Labels:  controller-uid=2ec4494f-8035-11e8-a389-0800270c281a
           job-name=primes-parallel
  Containers:
   primes:
    Image:      ubuntu
    Port:       <none>
    Host Port:  <none>
    Command:
      bash
    Args:
      -c
      current=0; max=70; echo 1; echo 2; for((i=3;i<=max;)); do for((j=i-1;j>=2;)); do if [  `expr $i % $j` -ne 0 ] ; then current=1; else current=0; break; fi; j=`expr $j - 1`; done; if [ $current -eq 1 ] ; then echo $i; fi; i=`expr $i + 1`; done
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  1m    job-controller  Created pod: primes-parallel-b7d4s
  Normal  SuccessfulCreate  1m    job-controller  Created pod: primes-parallel-rwww4
  Normal  SuccessfulCreate  54s   job-controller  Created pod: primes-parallel-kfpqc
```
As you see, all three pods successfully completed the task and exited. We can verify that the Job was completed as well by running:

```
kubectl get jobs/primes-parallel
NAME              DESIRED   SUCCESSFUL   AGE
primes-parallel   3         3            6m
``` 
If you look into the logs of each pod, you’ll see that each of them completed the prime numbers calculation successfully (to check logs, take the pod name from the Job description):
```
kubectl logs primes-parallel-b7d4s
1
2
3
5
7
11
13
17
19
23
29
31
```
```
kubectl logs primes-parallel-rwww4
1
2
3
5
7
11
13
17
19
23
29
31
```
 
That’s it! Parallel jobs with fixed completions count are very useful when you want to perform the same task multiple times. However, what about a scenario when we want one task to be completed in parallel by several pods? Enter parallel jobs with a work queue!

Parallel Jobs with a Work Queue
Parallel jobs with a work queue can create several pods which coordinate with themselves or with some external service which part of the job to work on. If your application has a work queue implementation for some remote data storage, for example, this type of Job can create several parallel worker pods that will independently access the work queue and process it. Parallel jobs with a work queue come with the following features and requirements:

for this type of Jobs, you should leave .spec.completions  unset.
each worker pod created by the Job is capable to assess whether or not all its peers are done and, thus, the entire Job is done (e.g each pod can check if the work queue is empty and exit if so).
when any pod terminates with success, no new pods are created.
once at least one pod has exited with success and all pods are terminated, then the job completes with success as well.
once any pod has exited with success, other pods should not be doing any work and should also start exiting.
Let’s add parallelism to the previous Job spec to see how this type of Jobs work:
```
apiVersion: batch/v1
kind: Job
metadata:
  name: primes-parallel-2
  labels:
    app: primes
spec:
  parallelism: 3
  template:
    metadata:
      name: primes
      labels:
        app: primes
    spec:
      containers:
      - name: primes
        image: ubuntu
        command: ["bash"]
        args: ["-c",  "current=0; max=110; echo 1; echo 2; for((i=3;i<=max;)); do for((j=i-1;j>=2;)); do if [  `expr $i % $j` -ne 0 ] ; then current=1; else current=0; break; fi; j=`expr $j - 1`; done; if [ $current -eq 1 ] ; then echo $i; fi; i=`expr $i + 1`; done"]
      restartPolicy: Never
```
The only difference from the previous spec is that we omitted .spec.completions  field, added the .spec.parallelism  field and set its value to 3.

Now, let’s open two terminal windows as in the previous example. In the first terminal, watch the pods:
``
kubectl get pods -l app=primes -w
 ```
Let’s save the spec in the job-prime-3.yaml  and create the job in the second terminal:

```
kubectl create -f job-prime-3.yaml
job.batch "primes-parallel-2" created
``` 
Next, let’s see what’s happening in the first terminal window:

```
kubectl get pods -l app=primes -w
NAME                      READY     STATUS    RESTARTS   AGE
primes-parallel-2-b2whq   0/1       Pending   0          0s
primes-parallel-2-b2whq   0/1       Pending   0         0s
primes-parallel-2-vhvqm   0/1       Pending   0         0s
primes-parallel-2-cdfdx   0/1       Pending   0         0s
primes-parallel-2-vhvqm   0/1       Pending   0         0s
primes-parallel-2-cdfdx   0/1       Pending   0         0s
primes-parallel-2-b2whq   0/1       ContainerCreating   0         0s
primes-parallel-2-vhvqm   0/1       ContainerCreating   0         0s
primes-parallel-2-cdfdx   0/1       ContainerCreating   0         0s
primes-parallel-2-b2whq   1/1       Running   0         4s
primes-parallel-2-cdfdx   1/1       Running   0         7s
primes-parallel-2-vhvqm   1/1       Running   0         10s
primes-parallel-2-b2whq   0/1       Completed   0         17s
primes-parallel-2-cdfdx   0/1       Completed   0         21s
primes-parallel-2-vhvqm   0/1       Completed   0         23s
``` 
As you see, the kubectl  created three pods simultaneously. Each pod was calculating the prime numbers in parallel and once each of them completed the task, the Job was successfully completed as well.

Let’s see more details in the Job description:

```
kubectl describe jobs/primes-parallel
Name:           primes-parallel
Namespace:      default
Selector:       controller-uid=d8bfbf9c-8038-11e8-a389-0800270c281a
Labels:         controller-uid=d8bfbf9c-8038-11e8-a389-0800270c281a
                job-name=primes-parallel
Annotations:    <none>
Parallelism:    3
Completions:    <unset>
Start Time:     Thu, 05 Jul 2018 12:50:27 +0300
Pods Statuses:  0 Running / 3 Succeeded / 0 Failed
Pod Template:
  Labels:  controller-uid=d8bfbf9c-8038-11e8-a389-0800270c281a
           job-name=primes-parallel
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
  Normal  SuccessfulCreate  1m    job-controller  Created pod: primes-parallel-8ggnq
  Normal  SuccessfulCreate  1m    job-controller  Created pod: primes-parallel-gbwgm
  Normal  SuccessfulCreate  1m    job-controller  Created pod: primes-parallel-66w65
```
As you see, all three pods succeeded in performing the task. You can also check the pods’ logs to see the calculation results:
```

kubectl logs primes-parallel-8ggnq
1
2
3
5
7
11
```
```
kubectl logs primes-parallel-gbwgm
1
2
3
5
7
11
```
```
kubectl logs primes-parallel-66w65
1
2
3
5
7
11
```
 
From the logs above, it may look like our Job with a work queue acted the same way as the job with a fixed completions count. Indeed, all three pods created by the Job calculated prime numbers in the range of 1-110. However, the difference is that in this example all three pods did their work in parallel. If we had created a work queue for our worker pods and some script to process items in that queue, we could make the pods access different batches of numbers (or messages, emails etc.) in parallel until no items left in the queue. In this example, we don’t have a work queue and a script to process it. That’s why all three pods created by the Job did the same task to completion. However, this example is enough to illustrate the main feature of this type of Jobs – parallelism.

K8s Jobs

 

In the real-world scenario, we could imagine a Redis list with some work items (e.g messages, emails) in it and three parallel worker pods created by the Job (see the Image above). Each pod could have a script to requests a new message from the list, process it, and check if there are more work items left. If no more work items exist in the list, the pod accessing it would exit with success telling the controller that the work was successfully done. This notification would cause other pods to exit as well and the entire job to complete. Given this functionality, parallel jobs with a work queue are extremely powerful in processing large volumes of data with multiple workers doing their tasks in parallel.

Cleaning Up
As our tutorial is over, let’s clean up all resources:

Delete the Jobs

Deleting a Job will cause all associated pods to be deleted as well.
```
kubectl delete job primes-parallel-2
job.batch "primes-parallel-2" deleted
kubectl delete job primes-parallel
job.batch "primes-parallel" deleted
kubectl delete job primes
job.batch "primes" deleted
 ```
