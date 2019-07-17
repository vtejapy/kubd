## Deployments 

- Deployments declaration in kubernetes allows you to do app deployments and updates
- when using the deployment object, you define the state of your application
  - kubernetes will then make sure the clusters matches your desired state
- just using the replication controller or replication set might be cumbersome to deploy apps

- with a deployment object you can : 

  - create a deployment(eg: deploying an app)
  - update a deployment(eg: deploying a new version)
  - Do rolling updates (zero downtime deployments)
  - Roll back to a previous version
  - pause/resume a deployment(eg: roll-out to only a certain percentage)

  <<exampl>>
