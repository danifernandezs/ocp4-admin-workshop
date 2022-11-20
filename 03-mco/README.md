# Review OpenShift Configuration

## MachineSets and Machines

The `MachineSet` defines a desired state for a set of `Machine` objects.

When using IPI installations, then, there is an `Operator` whose job it is to make
sure that there is actually an underlying instance for each `Machine` and,
finally, that every `Machine` becomes a `Node`.

````bash
oc get machineset -n openshift-machine-api
````

When OpenShift was installed, the installer interrogated the cloud provider
to learn about the available AZs (since this is on AWS). It then ultimately
created a `MachineSet` for each AZ and then scaled those sets, in order,
until it reached the desired number of `Machines`. Since the default
installation has 3 workers, the first 3 AZs got one worker each. The rest got
zero.

````bash
oc get machine -n openshift-machine-api
````

Each `Machine` has a corresponding `INSTANCE`.

## Nodes

Verify that all nodes on your cluster are ready:

````bash
oc get nodes
````

Verify whether any of your nodes are close to using all of the CPU and memory available to them.

Repeat the following command a few times to prove that you see actual usage of CPU and memory from your nodes. The numbers you see should change slightly each time you repeat the command.

````bash
oc adm top node
````

Use the oc describe command to verify that all of the conditions that might indicate problems are false.

Spend sometime to understand all the information that is shown.

````bash
oc describe node NODENAME
````

Follow the logs of the Kubelet from the same node that you inspected for CPU and memory usage in the previous step. 

````bash
oc adm node-logs --tail 1 -u kubelet ip-10-0-137-166.eu-west-1.compute.internal
----

Start a shell session to the same node that you previously used to inspect its OpenShift services and pods. Do not make any change to the node, such as stopping services or editing configuration files.

Start a shell session on the node, and then use the chroot command to enter the local file system of the host.

````bash
oc debug node/NODENAME
````

Still using the same shell session, verify that the Kubelet and the CRI-O container engine are running. Type q to exit the command.

````bash
systemctl status kubelet
````

````bash
systemctl status crio.service
````

### Cluster Scaling

Because of the magic of `Operators` and the way in which OpenShift uses them
to manage `Machines` and `Nodes`, scaling your cluster in OpenShift 4 is
extremely trivial.

Look at the list of `MachineSets` again:

````bash
oc get machineset -n openshift-machine-api
````

Within that list, we will scale one of the `MachineSet` objects with the
`oc scale` command. Run:

````bash
CLUSTERNAME=$(oc get  infrastructures.config.openshift.io cluster  -o jsonpath='{.status.infrastructureName}')
ZONENAME=$(oc get nodes -L topology.kubernetes.io/zone  --no-headers  | awk '{print $NF}' | tail -1)
oc scale machineset ${CLUSTERNAME}-worker-${ZONENAME} -n openshift-machine-api --replicas=2
````

Take special note the `MachineSet` scaled is likely different from
the one that is shown in the lab guide. You should see a note that the
`MachineSet` was successfully scaled. Now, look at the list of `Machines`:

````bash
oc get machines -n openshift-machine-api
````

You probably already have a new entry for a `Machine` with a `STATE` of
`Pending`.

At this point, in the background, the bootstrap process is happening
automatically. After several minutes (up to five or so), take a look at the
output of:

````bash
oc get nodes
````

You should see your fresh and happy new node as the one with a very young age:


````
It can take several minutes for a `Machine` to be prepared and added
as a `Node`. You can follow the process by running a `watch` against
`oc get nodes` if you wish.
````
