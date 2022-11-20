## Scaling an Application

### Specifying Pod Replicas in Configuration Workloads

The number of pod replicas for a specific deployment can be increased or decreased to meet your needs. Despite the `ReplicaSet` and `ReplicationController` resources, the number of replicas needed for an application is typically defined in a deployment resource. A replica set or replication controller (managed by a deployment) guarantees that the specified number of replicas of a pod are running at all times. The replica set or replication controller can add or remove pods as necessary to conform to the desired replica count.

Deployment resources contain:

* The desired number of replicas
* A selector for identifying managed pods
* A pod definition, or template, for creating a replicated pod (including labels to apply to the pod)

Create a new project for this guided exercise named schedule-scale.

````
oc new-project schedule-scale
````

Deploy a test application for this exercise which explicitly requests container resources for CPU and memory.

````
oc apply -f 01-deployment.yaml -n schedule-scale
````

Verify that your application pod has a status of Running. You might need to run the oc get pods command multiple times.

````
oc get pods
````

Verify that your application pod specifies resource limits and requests.

````
oc describe pod/pod | grep -A2 -E "Limits|Requests"
````

Manually scale the deployment by first increasing and then decreasing the number of running pods.

Scale the deployment up to five pods.

````
oc scale --replicas 5 deployment/scale
````

Verify that all five application pods are running. You might need to run the oc get pods command multiple times.

Scale the deployment back down to one pod.

````
oc scale --replicas 1 deployment/scale
````

Configure the application to automatically scale based on load, and then test the application by applying load.


Create a horizontal pod autoscaler that ensures the loadtest application always has 2 application pods running; that number can be increased to a maximum of 10 pods when CPU load exceeds 50%.

````
oc autoscale deployment/scale --min 2 --max 10 --cpu-percent 50
````

Wait until the loadtest horizontal pod autoscaler reports usage in the TARGETS column.

````
oc get hpa/scale
````

### Clean Up

````
oc delete project schedule-scale
````
