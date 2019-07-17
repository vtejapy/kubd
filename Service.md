# services 

- pods are very dynamics, they come and go on the kubernetes cluster
  - when using a replication controller , pods are terminated and created duting scalling operations
  - when using deployments, when updaeting the image version, pods are terminated and new pods take the place of older pods
- that's why pods should never be accessed directly, but alwalys through a service
- a service is the logical bridge between the "mortal" pods and other services or end-users

- when using the "kubectl expose" comamnds , created a new service for your pod , so it could be accessed externally 

- creating a service wil create an endpoint for your pod(s):

  - a clusterIp: virtual Ip address only reachable from within the cluster (this is default)
  - a NodePort: a port that is the same on each node that is also reachable externally
  - a LoadBalancer: a LoadBalancer created by the cloud provider that wil route externally traffic to every node on the NodePort(ELB on AWS)

- the options just show only allow you to create virtual IP's or ports 
- there is also a possibility to use DNS names
  - ExternalName can provide a DNS name for the service
  - e.g: for service discovery using DNS
  - this only works when the DNS add-on is enabled

- NOte: by default service can only run between ports 30000-32767, but you could change this behaviour by adding the --service-node-port-range=argumen
to the kube-apiserver
