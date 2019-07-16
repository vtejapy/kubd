### configMap
- configuration parameter that are not secret, can be put in a configMap
- the input is again key-value pairs
- the configMap key-value pairs can then be read by the app using: 
  - Environment variables
  - Container commandline arguments in the pod configuration
  - using volumes
- A configMap can also contain full configuraion files
   - an webserver config file
- this file can then be mounted using volumes where the application expects its config file
- this way you can "inject" configuraion setting into containers without changing the container itself
nginx.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: helloworld-nginx
  labels:
    app: helloworld-nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.11
    ports:
    - containerPort: 80
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/conf.d
  - name: k8s-demo
    image: wardviaene/k8s-demo
    ports:
    - containerPort: 3000
  volumes:
    - name: config-volume
      configMap:
        name: nginx-config
        items:
        - key: reverseproxy.conf
          path: reverseproxy.conf
```
nginx-service.yml
```
apiVersion: v1
kind: Service
metadata:
  name: helloworld-nginx-service
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: helloworld-nginx
  type: NodePort
```
reverseproxy.conf
```
server {
    listen       80;
    server_name  localhost;

    location / {
        proxy_bind 127.0.0.1;
        proxy_pass http://127.0.0.1:3000;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```
