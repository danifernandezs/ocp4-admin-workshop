# Cluster Network Ingress

## Default IngressController in OCP4

In the OCP 3.x version, the router sharding was implemented with patching directly the routers with the “oc adm router” commands for create/manage the routers and their labels, to apply the namespace and route selectors.

But in OCP 4.x, the rules of the game changed: the OCP Routers (and other elements including LBs, DNS entries, etc) are managed by the OpenShift Ingress Operator.

Ingress Operator is an OpenShift component which enables external access to cluster services by configuring Ingress Controllers, which route traffic as specified by OpenShift Route and Kubernetes Ingress resources. Furthermore, the Ingress Operator implements the OpenShift ingresscontroller API.

Into every new OCP4 cluster, the ingresscontroller “default” is deployed into the openshift-ingress-operator namespace:

````
oc get ingresscontroller -n openshift-ingress-operator
````

As we can see into the ingresscontroller “default”, a ingress controller with replica 2 is deployed. Also pay attention that the spec definition is empty (spec: {}).

````
oc get ingresscontroller default -n openshift-ingress-operator -o yaml
````

````
oc get pod -n openshift-ingress
````

## Router sharding - Adding Ingress Controller for internal and production traffic application routes using routeSelector

In several cases, customers want to have isolated DMZ traffic routes of their applications, from the internal traffic application routes (for example, only reachable from inside the Private subnet of the cluster). And also it's quite a common practice to create logical seggregations for different environments (e.g. internal/development and production)


````
oc apply -f 01-internal.yaml 
oc apply -f 02-public.yaml 
````

And check that are created in the cluster:

````
oc get ingresscontroller -n openshift-ingress-operator
````

Furthermore, two replicas of the new brand ingresses controllers appears in the openshift-ingress namespace:

````
oc get pod -n openshift-ingress
````

### Testing the Route Sharding I - internal and production application routes

Create a new project and deploy an application for testing purposes (in this case, we use the django-psql-example):

````
oc new-project test-sharding
oc new-app django-psql-example
````

The new-app deployment creates two pods, a django frontend and a postgresql database, and also a Service and a Route:

````
oc get po
````

````
oc get route -n test-sharding
````

This route is exposed by default to the “router-default”, using the *apps. domain route.

Let's tweak the route, and add the label that matches to the routeSelector defined into our internal ingresscontroller:

````
metadata:
  labels:
    router: internal
````

With a describe of the route, check that the route is created correctly:

````
oc delete route django-psql-example -n test-sharding
````

Let's tweak the route, and add the label that matches to the routeSelector defined into our public ingresscontroller:

````
metadata:
  labels:
    router: public
````

With a describe of the route, check that the route is created correctly:

Create the DNS record pointing to the public Router.

### Clean-up

````
oc delete project test-sharding
oc delete ingresscontroller internal public -n openshift-ingress-operator
oc patch ingresscontroller/default -n openshift-ingress-operator --type json -p '[{ "op": "remove", "path": "/spec/routeSelector"}]'
````

### Source Links

* https://rcarrata.com/openshift/ocp4_route_sharding/[Deep dive of Route Sharding in OpenShift 4]
