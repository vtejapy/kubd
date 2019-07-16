# Secrets
- secrets provides a way in kubernetes to distribute credentials,keys,password or "secret" data to the pods

- kubernetes itself uses this Secrets mechanism to provide the credentials to access the internal API
- you can also use the same mechanism to provide secrets to your application

- secrets is one way to provide secrets, native to kubernetes 

   - there are stil other ways your container can get its secrets if you don't want to use Secrets (eg: using an external valt service in your app)

***Secrets can be used in the following ways***:

  - use secrets as environment variables
  - use secrets as a file in a pod
     - this step uses volume to be mounted in a container
     - in this volume you have files
     - can be used for instance for dotenv files or your app can just read this file
 - use an external image to pull secrets(from a private image registry)
 helloworld-secrets.yml
 ```
apiVersion: v1
kind: Secret
metadata:
  name: db-secrets
type: Opaque
data:
  username: cm9vdA==
  password: cGFzc3dvcmQ=
 ```
helloworld-secrets-volumes.yml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: helloworld-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - name: k8s-demo
        image: wardviaene/k8s-demo
        ports:
        - name: nodejs-port
          containerPort: 3000
        volumeMounts:
        - name: cred-volume
          mountPath: /etc/creds
          readOnly: true
      volumes:
      - name: cred-volume
        secret: 
          secretName: db-secrets
```
