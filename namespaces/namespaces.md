# NameSpaces 

## Why?
- Structure our components.
- Avoid conflicts between teams.
- Share resources between different environments.
- Access and resource limitting on namespace level.

## Remember!
- We can't access most of the resources from another namespace.
    - For example if namespace_A has a configMap conatining info about database in some other namespace, namespace_B won't be able to connect to that database by using configMap from namespace_A since it can't access it. The same goes for secrets.
- Shared resources can be accessed (nginx, elastic search).
    - We can however access service from another namespace. For example the configMap in namespace_A will have the database_url field from a service in the database namespace.
    ```
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: mongodb-configmap
    data:
      database_url: mongodb-service.database # The name of the service and it's namespace.
    ```
- Some components cannot be created in a nampespace. They live globally in a cluster.
    - Persisten volume 
    - node
    ```
    $ kubectl api-resources --namespaced=false
    ```
- Resources are assigned to the default namespace in case one isn't provided.

## Usage
```
$ kubectl apply -f mysql-configmap.yaml ---namespace=my-namespace
```
The other way is through the config file itslef.
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-configmap
  namespace: my-namespace
data:
  database_url: mysql-service.database
```

## Change the default namespace
No solution by default.
Install kubens
```
$ yay -S kubectx
```
This will install kubens too.

Check the available namespaces.
```
$ kubens
```
Change the active namespace.
```
$ kubens my-namespace
```
