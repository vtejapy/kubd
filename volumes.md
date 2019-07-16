## volumes:
- volumes in kubernetes allow you to store data outside the container
- when a container stops, all data on the container itself is lost 
   * that's why up until now I've been using stateless apps: apps that dont keep a local state , but store their state in an external service
   *  external service like a database, caching server (eg: MySQL,AWS,S3)
- persistent volumes in kubernetes allow you attach a volume to a container that will exists even when the container stops 
* using volumes, you could deploy application with state on your cluster 
  *  those application need to read/write to files on the local filesystem that need to be persistent in time
* you could run a mysql database using persistent volumes 
* if your node stop working, the pod can be resheduled on another node, and the volume can be attached to the new node.
