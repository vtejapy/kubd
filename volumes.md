## volumes:
- volumes in kubernetes allow you to **store data outside the container**
- when a container **stops**, all data on the container itself is **lost** 
   * that's why up until now I've been using **stateless** apps: apps that dont keep a **local** state , but store their state in an **external service**
   *  external service like a database, caching server (eg: MySQL,AWS,S3)
- persistent volumes in kubernetes allow you **attach a volume** to a container that will **exists** even when the **container** stops 
* using volumes, you could deploy **application with state** on your cluster 
  *  those application need to read/write to files on the **local filesystem** that need to be persistent in time
* you could run a **mysql** database using persistent volumes 
* if your **node stop**  working, the pod can be resheduled on another node, and the volume can be attached to the new node.
### Adv of kubernetes volumes vs docker volumes
   -  containers has access to volume
   - volumes are associated with lifecycle of pod
   - supports many types of volumes

### volumes types
- Ephermal: same lifetime as pods
-  Durable : Beyond pods lifetime


**emptyDir**
 - creates empty directory first created when a pod is assigned to a node
  -  stays as long as pod is running
  -  once pods is removed from a node, emptyDir is deleted forever

***Use cases***:
 -  Temporary space

**hostPath**:
 - mounts a file or directory from the host node's filesystem into your pods
  -  remains even afer the pod is terminated
  -  similar to docker volume
  -  Host issues might cause problem to hostpath

**gcePersistentDisk**
-  volumes mounts a google compute Engine (GCE) persistent Disk into pod
-  volume data is persisted pods termination
-  Read-Write only on one node and Read-Write on many nodes
***Restrications***:
 -  you must create a pd using gcloud or the GCE API or UI befor can do use it 
 - the nodes on which pods are running must be GCE vms
- those VMs need to be in the same GCE project and zone as the PD
