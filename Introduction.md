### what is kubernetes ?
Kubernetes (K8s) is an open-source system for automating deployment, scaling, and management of containerized applications.

- kubernetes is an opensource orchestration system for Docker Containers
  - It lets you schedule containers on a cluster of machines 
  - you can run multiple containers on one machine
  - you can run long running services (like web applications)
  - kubernetes will manage the state of these containers
    - can start the container on specific nodes
    -  will restart a container when it gets killed
    -  can move containers from one node to another node
 - instead of just running a few docker containers on one host manually kubernetes is a platform that will manage the containers for you 
 - kubernetes cluster can start with one node until thousands of nodes 
 - some other popular docekr orchestrators are 
    - Docker swarm
    - Apache mesos
### Advantages

- you can run kubernetes anywhere:
  - on-premise(own datacenter)
  - public (google coud, aws)
  - Hybrid: public & private
- highly modular
- opensource
- great community
- backed by google


# Architecure

- kubernetes cluster is split into 2 parts

  - control plane(master)
  - The (worker) nodes

### componentes of control plane
- The etcd distributed presistent storege
- The ApI server
- the scheduler
- the controller manager

<<diagram1>>


# components of The worker nodes

- the kublet
- the kubernetes service proxy (kube-proxy)
- the container Runtime(Docker,rkt or others)
- pod

kube-apiserver:
-->Component on the master that exposes the Kubernetes API. It is the front-end for the Kubernetes control plane.

-->It is designed to scale horizontally – that is, it scales by deploying more instances. See Building High-Availability Clusters.

etcd:
Consistent and highly-available key value store used as Kubernetes’ backing store for all cluster data.

If your Kubernetes cluster uses etcd as its backing store, make sure you have a back up plan for those data.

kube-scheduler:
Component on the master that watches newly created pods that have no node assigned, and selects a node for them to run on.

Factors taken into account for scheduling decisions include individual and collective resource requirements, hardware/software/policy constraints, affinity and anti-affinity specifications, data locality, inter-workload interference and deadlines.

kube-controller-manager
Component on the master that runs controllers .

Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.

These controllers include:

Node Controller: Responsible for noticing and responding when nodes go down.
Replication Controller: Responsible for maintaining the correct number of pods for every replication controller object in the system.
Endpoints Controller: Populates the Endpoints object (that is, joins Services & Pods).
Service Account & Token Controllers: Create default accounts and API access tokens for new namespaces.
cloud-controller-manager
cloud-controller-manager runs controllers that interact with the underlying cloud providers. The cloud-controller-manager binary is an alpha feature introduced in Kubernetes release 1.6.

cloud-controller-manager runs cloud-provider-specific controller loops only. You must disable these controller loops in the kube-controller-manager. You can disable the controller loops by setting the --cloud-provider flag to external when starting the kube-controller-manager.

cloud-controller-manager allows the cloud vendor’s code and the Kubernetes code to evolve independently of each other. In prior releases, the core Kubernetes code was dependent upon cloud-provider-specific code for functionality. In future releases, code specific to cloud vendors should be maintained by the cloud vendor themselves, and linked to cloud-controller-manager while running Kubernetes.

The following controllers have cloud provider dependencies:

Node Controller: For checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding
Route Controller: For setting up routes in the underlying cloud infrastructure
Service Controller: For creating, updating and deleting cloud provider load balancers
Volume Controller: For creating, attaching, and mounting volumes, and interacting with the cloud provider to orchestrate volumes
Node Components
Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.

kubelet
An agent that runs on each node in the cluster. It makes sure that containers are running in a pod.

The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. The kubelet doesn’t manage containers which were not created by Kubernetes.

kube-proxy
kube-proxy is a network proxy that runs on each node in the cluster.

It enables the Kubernetes service abstraction by maintaining network rules on the host and performing connection forwarding.

kube-proxy is responsible for request forwarding. kube-proxy allows TCP and UDP stream forwarding or round robin TCP and UDP forwarding across a set of backend functions.

Container Runtime
The container runtime is the software that is responsible for running containers.

Kubernetes supports several container runtimes: Docker, containerd, cri-o, rktlet and any implementation of the Kubernetes CRI (Container Runtime Interface).

Addons
Addons are pods and services that implement cluster features. The pods may be managed by Deployments, ReplicationControllers, and so on. Namespaced addon objects are created in the kube-system namespace.

Selected addons are described below, for an extended list of available addons please see Addons.

DNS
While the other addons are not strictly required, all Kubernetes clusters should have cluster DNS, as many examples rely on it.

Cluster DNS is a DNS server, in addition to the other DNS server(s) in your environment, which serves DNS records for Kubernetes services.

Containers started by Kubernetes automatically include this DNS server in their DNS searches.

Web UI (Dashboard)
Dashboard is a general purpose, web-based UI for Kubernetes clusters. It allows users to manage and troubleshoot applications running in the cluster, as well as the cluster itself.

Container Resource Monitoring
Container Resource Monitoring records generic time-series metrics about containers in a central database, and provides a UI for browsing that data.

