### health check:

- if your applicatoin malfunctions, the pod and container can still be running, but the application might not work anymore
- to detect and resolve problems with your application, you can run health
checks
-  you can run 2 difference types of health checks
   - running a command in the container periodically 
   - periodic checks on a URL (HTTP)
- the typical production behind a load balancer should alwalys have health checks implemented in some way to ensuer availability and resiliency of the app
