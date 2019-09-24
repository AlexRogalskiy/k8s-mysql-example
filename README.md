# k8s-mysql-example

An example mysql deployment for Kubernetes (or OpenShift)

## Caveat

This is an example showing a simple deployment of mysql not intended for production use. In particular, the deployment does not make any high availability or redundancy guarantees which a productive deployment would need to make. We are using a single replica, launched using a `Deployment` whereas in real life we would be using a `StatefulSet` configuration.

## Deployment

We will provide the examples using `kubectl`, the command line interface for plain Kubernetes. You can derive the OpenShift variant by replacing `kubectl` with `oc`, the command line interface for OpenShift in all examples below.

### Clone the repo

Clone the repo using git (or download it) and make sure you are inside the repo's root directory.

    ```
    git clone https://github.com/mkretz/k8s-mysql-example.git
    cd k8s-mysql-example
    ```

### Create a secret

We will use a secret to store the database credentials. The secret is not contained in the yaml files, since it contains things which should not end up in source control. We use example credentials below. Of course, you should use secure ones.

    ```
    kubectl create secret generic mysql-example-secret --from-literal=mysqlRootPassword='admin' --from-literal=mysqlDatabase='example' --from-literal=mysqlUser='exampleuser' --from-literal=mysqlPassword='examplepwd'
    ```

### Create the persistent volume claim

Next we need to create a persistent volume claim in order for the data to be stored persistently.

```
kubectl apply -f pvc.yaml
```

### Create the persistent volume claim

Next we need to deploy mysql.

```
kubectl apply -f deployment.yaml
```

### Create a service

Last but not least we need to expose mysql as a service in the cluster, so that other components can access it.

```
kubectl apply -f service.yaml
```

## Testing the deployment

In order to test the deployment we will access the database in the same way as a component in the cluster would. To this end we start a mysql client as a temporary pod in the cluster, making use of the credentials we defined in the secret.

```
kubectl run -it --rm --image=mysql:8 --restart=Never mysql-client -- mysql -h mysql -uexampleuser -pexamplepwd
```

After a short while we should get the mysql prompt, where we can test creating a table for example.

```
mysql> use example;
Database changed

mysql> create table if not exists `example`.`Test` (`email` VARCHAR(255));
Query OK, 0 rows affected (0.03 sec)

mysql> show tables;
+-------------------+
| Tables_in_example |
+-------------------+
| Test              |
+-------------------+
1 rows in set (0.00 sec)

mysql>
```

This shows us that the database is working as it should. We can now use the host name `mysql` to access the database from other components.
