````
oc label node node-name node-role.kubernetes.io/infra=
````

````
oc apply -f 01-infra-mcp.yaml
````

````
oc edit ingresscontroller default -n openshift-ingress-operator -o yaml
````

````
spec:
  nodePlacement:
    nodeSelector:
      matchLabels:
        node-role.kubernetes.io/infra: ""
````

````
./butane-amd64 02-ntp-worker.yaml -o 99-worker-chrony.yaml
````

````
oc apply -f 99-worker-chrony.yaml
````

````
oc get mc
````
