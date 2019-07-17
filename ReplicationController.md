## replication controller

### Scaling:

- if your application is stateless you can horizontally scale it
  - stateless = your application doesn't have a state , it doesn't write any local files/keeps local sessions
  - all traditional databases (mysl, postgres) are stateful, they hav database files that can't split over multiple instances
- most web applications can be made stateless
  - session management needs to be done outside the container
  - any files that need to be saved can't be saved locally on the container
- the sateful apps can;t horizontally scale, but you can run them in a single container and vertically scale(allocate memory cpu/memory/Disk)

- scaling in kubernetes can be done using the Replication Controller
- the replication controller will ensuer a specified number of pod replicas will run at all time.

- a pod created with the replica controller will automatically be replaced if they fail, get deleted , or terminated
- using the replication controller is also recommended if you just want to make sure 1 pod is alwalys running, even after reboot
  - you can then run a replicatoin controller with just 1 replica
  - this makes sure that the pod is alwalys running


syntax
