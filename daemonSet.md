## daemonset

- daemon sets ensure that every single node in the kubernetes cluster runs the same pod resource
   - this is useful if you want to ensure that a certain pod is running on every single kubernetes node
- when a node is added to the cluster , a new pod will be started automatically 
- same when a node is removed. the pod wil not be resheduled on another node


***typical use cases***: 
   - Logging aggregators 
   - Monitoring
   - Load Balancers/Reverse Proxies/Api Gateways
   - Running a daemon that only needs one instance per physical instance

demonset_ex.yml

```
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: nginx-daemonset
spec:
  selector:
    matchLabels:
      demotype: nginx-daemonset-demo
  template:
    metadata:
      labels:
        demotype: nginx-daemonset-demo
    spec:
      containers:
      - image: nginx
        name: nginx
```
