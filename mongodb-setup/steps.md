
# Create a mongodb deployment

## (1) Create a deployment file for mongodb.
```mongodb-deployemnt.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:latest
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef: 
              name: mongodb-secret
              key: mongo-root-username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef: 
              name: mongodb-secret
              key: mongo-root-password
```
## (2) Create a secret.
A secret is used for storing sensitive data like usernames and passwords.
```mongo-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
type: Opaque
data:
  mongo-root-username: YWRtaW4=  # Base64-encoded value of the username 'admin'
  mongo-root-password: cGFzc3dvcmQ=  # Base64-encoded value of the password 'password'
```

The secret must be before the deployment since the deployment refers to it.
```bin/bash
$ kubectl apply -f mongo-secret.yaml
```
Verify.
```bin/bash
$ kubectl get secret
```
## (3) Create an internal service
Add the following to the end of mongodb-deployemnt.yaml
```
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
spec:
  selector:
    app: mongodb
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
```
Create the service.
```
$ kubectl apply -f mongo-secret.yaml
```
Get the service.
```
$ kubectl get service
```
To verify make sure the endpoint in the service description matches the pod url
```
$ kubectl describe service mongodb-service
$ kubectl get pod -o wide
```
## (4) Setup mongo-express
We need to tell the mongo-express app which database to connect to? Internal service
Which creds to authenticate with?

We store the mongodb server url in a config map so other applications can also use it.
Create a configuration file for config map.
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
data:
  database_url: mongodb-service # The name of the service
```
Create the config map
```
$ kubectl apply -f mongo-configmap.yaml
$ kubectl get configmap
```
Now create the mongo express deployment configuration.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express-deployment
  labels:
    app: mongo-express
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo-express
  template:
    metadata:
      labels:
        app: mongo-express
    spec:
      containers:
      - name: mongo-express
        image: mongo-express:latest
        ports:
        - containerPort: 8081
        env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          valueFrom:
            secretKeyRef: 
              name: mongodb-secret
              key: mongo-root-username
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD
          valueFrom:
            secretKeyRef: 
              name: mongodb-secret
              key: mongo-root-password
        - name: ME_CONFIG_MONGODB_SERVER
          valueFrom:
              configMapKeyRef: 
                name: mongodb-configmap
                key: database_url
```
```
$ kubectl apply -f mongo-express.deployment.yaml
$ kubectl get all | grep mongo-express
```
## (5) Create an external service
At the end of the configuration file for mongo-express deployemnt add the following code:
```
---
apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service
spec:
  selector:
    app: mongo-express
  type: LoadBalancer # Makes it an external service.   
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
      nodePort: 30000 # Port for external ip address & we put this in the browser
```
In the above, the "nodePort" is required because it is the port for external ip address.
External services have both an internal ip address and an external ip address, so we need ports for both of them.
```
$ kubectl apply -f mongo-express.deployment.yaml
$ kubectl get service
```
## (6) Assign an extrenal ip address (minikube).
```
$ minikube service mongo-express-service
```

