# OpenShift Network Policy Based SDN
OpenShift's default software defined network (SDN) inside the platform is based
on Open vSwitch and is called OpenShiftSDN.

Moving forward, OVNKubernetes is the perfered SDN to use on OpenShift, and will be the
default in future releases. OVNKubernetes is backwards compatible
with OpenShiftSDN. To read more about OVNKubernetes, please read
link:https://docs.openshift.com/container-platform/4.11/networking/ovn_kubernetes_network_provider/about-ovn-kubernetes.html[the official documentation].

Kubernetes `NetworkPolicy` allows projects to truly isolate their
network infrastructure inside OpenShift’s software defined network. While you
have seen projects isolate resources through OpenShift’s RBAC, the network policy
SDN plugin is able to isolate pods in projects using pod and namespace label selectors.

## Execute the Creation Script
````
Only users with project or cluster administration privileges can manipulate *Project*
network policies.
````

Execute a script that will create two
*Projects* and then deploy a *Deployment* with a *Pod* for you:

````
./create-net-projects.sh
````

## Examine the created infrastructure
Two *Projects* were created for you, `netproj-a` and `netproj-b`. Execute the
following command to see the created resources:

````
oc get pods -n netproj-a
````

````
oc get pods -n netproj-b
````

We will run commands inside the pod in the `netproj-a` *Project* that will
connect to TCP port 5000 of the pod in the `netproj-b` *Project*.

### Test Connectivity (should work)
Now that you have some projects and pods, let's test the connectivity between
the pod in the `netproj-a` *Project* and the pod in the `netproj-b` *Project*.

To test connectivity between the two pods, run:

````
./test-connectivity.sh
````

You will see something like the following:


Note that the last line says `worked`. This means that the pod in the `netproj-a` *Project* was able to connect to the pod in the `netproj-b` *Project*.

This worked because, by default, the networkpolicy lets all pods in all projects can connect to each other.

## Restricting Access
With NetworkPolicy, it's possible to restrict access in a project by creating a `NetworkPolicy` custom resource (CR).

For example, the following restricts all access to all pods in a *Project* where this `NetworkPolicy` CR is applied. This is the equivalent of a `DenyAll` default rule on a firewall:

````
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: deny-by-default
spec:
  podSelector:
  ingress: []
````

````
oc apply -f deny-all.yaml -n netproj-b
````

````
oc get networkpolicy -n netproj-b
````

Note that the `podSelector` is empty, which means that this will apply to all pods in this *Project*. Also note that the `ingress` list is empty, which means that there are no allowed `ingress` rules defined by this `NetworkPolicy` CR.

To restrict access to the pod in the `netproj-b` *Project* simply apply the
above NetworkPolicy CR with:

### Test Connectivity #2 (should fail)
Since the "block all by default" `NetworkPolicy` CR has been applied, connectivity between the pod in the `netproj-a` *Project* and the pod in the `netproj-b` *Project* should now be blocked.

````
./test-connectivity.sh
````

Note the last line that says `FAILED!`. This means that the pod in the `netproj-a` *Project* was unable to connect to the pod in the `netproj-b` *Project* (as expected).

### Allow Access
With NetworkPolicy, it's possible to allow access to individual or groups of pods in a project by creating multiple
`NetworkPolicy` CRs.

The following allows access to port 5000 on TCP for all pods in the project
with the label `run: ose`. The pod in the `netproj-b` project has this label.

The ingress section specifically allows this access from all projects that
have the label `name: netproj-a`.

````
oc apply -f allow-projecta.yaml -n netproj-b
````

````
oc get networkpolicy -n netproj-b
````

Note that the `podSelector` is where the local project's pods are matched
using a specific label selector.

All `NetworkPolicy` CRs in a project are combined to create the allowed
ingress access for the pods in the project. In this specific case the "deny
all" policy is combined with the "allow TCP 5000" policy.

To allow access to the pod in the `netproj-b` *Project* from all pods in the
`netproj-a` *Project*, apply the above NetworkPolicy CR with:

### Test Connectivity #3 (should work again)
Since the "allow access from `netproj-a` on port 5000" NetworkPolicy has been applied,
connectivity between the pod in the `netproj-a` *Project* and the pod in the
`netproj-b` *Project* should be allowed again.

Test by running:

````
./test-connectivity.sh
````

## Clean-up
[source,bash,role="execute"]
````
oc delete project netproj-a netproj-b
````

## Links
* https://github.com/ahmetb/kubernetes-network-policy-recipes[Kubernetes Network Policy Recipes]
* https://rcarrata.com/openshift/egress-firewall/[Egress Firewall in OpenShift with OVN Kubernetes plugin]
* https://editor.cilium.io/[Network policy online editor]
