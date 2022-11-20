## OpenShift Log Aggregation
In this lab you will explore the logging aggregation capabilities of
OpenShift.

An extremely important function of OpenShift is collecting and aggregating
logs from the environments and the application pods it is running. OpenShift
ships with an elastic log aggregation solution: *EFK*. (**E**lasticSearch,
**F**luentd and **K**ibana)

The cluster logging components are based upon Elasticsearch, Fluentd, and
Kibana (EFK). The collector, Fluentd, is deployed to each node in the
OpenShift cluster. It collects all node and container logs and writes them to
Elasticsearch (ES). Kibana is the centralized, web UI where users and
administrators can create rich visualizations and dashboards with the
aggregated data. Administrators can see and search through all logs.
Application owners and developers can allow access to logs that belong to
their projects. The EFK stack runs on top of OpenShift.

### Deploying OpenShift Logging

OpenShift Container Platform cluster logging is designed to be used with the
default configuration, which is tuned for small to medium sized OpenShift
Container Platform clusters. The installation instructions that follow
include a sample Cluster Logging Custom Resource (CR), which you can use to
create a cluster logging instance and configure your cluster logging
deployment.

If you want to use the default cluster logging install, you can use the
sample CR directly.

If you want to customize your deployment, make changes to the sample CR as
needed. The following describes the configurations you can make when
installing your cluster logging instance or modify after installtion. See the
Configuring sections for more information on working with each component,
including modifications you can make outside of the Cluster Logging Custom
Resource.

#### Install the `Elasticsearch` and  `Cluster Logging` Operators in the cluster

In order to install and configure the `EFK` stack into the cluster,
additional operators need to be installed. These can be installed from the
`Operator Hub` from within the cluster via the GUI.

When using operators in OpenShift, it is important to understand the basics
of some of the underlying principles that make up the Operators.
`CustomResourceDefinion (CRD)` and `CustomResource (CR)` are two Kubernetes
objects that we will briefly describe.`CRDs` are generic pre-defined
structures of data. The operator understands how to apply the data that is
defined by the `CRD`. In terms of programming, `CRDs` can be thought as being
similar to a class. `CustomResource (CR)` is an actual implementations of the
`CRD`, where the structured data has actual values. These values are what the
operator will use when configuring it's service. Again, in programming terms,
`CRs` would be similar to an instantiated object of the class.

The general pattern for using Operators is first, install the Operator, which
will create the necessary `CRDs`. After the `CRDs` have been created, we can
create the `CR` which will tell the operator how to act, what to install,
and/or what to configure. For installing `openshift-logging`, we will follow
this pattern.

To begin, click on the `Console` tab or use the following link to log-in
to the OpenShift Cluster's GUI.
`{{ MASTER_URL }}`

Then follow the following steps:

1. Install the Elasticsearch Operator:

    1.1. In the OpenShift console, click `Operators` → `OperatorHub`.

    1.2. Type `Elasticsearch Operator` in the search field and click the `OpenShift Elasticsearch Operator` card from the list of available Operators, and then click `Install`.

    1.3. On the `Create Operator Subscription` page, select *Update Channel `stable`*, leave all other defaults
     and then click `Install`.

This makes the Operator available to all users and projects that use this
OpenShift Container Platform cluster.


----

The `Cluster Logging` operator needs to be installed in the
`openshift-logging` namespace. Please ensure that the `openshift-logging`
namespace was created from the previous steps

----

2. Install the Cluster Logging Operator:

    2.1. In the OpenShift console, click `Operators` → `OperatorHub`.

    2.2. Type `OpenShift Logging` in the search box and click the  `Red Hat OpenShift Logging` card from the list of available Operators, and click
    `Install`.

    2.3. On the `Install Operator` page, *select Update Channel `stable`*. Under ensure `Installation Mode` ensure that `A specific namespace on the cluster` is selected, and choose
     `Operator recommended Namespace: openshift-logging` under `Installed Namespace`. Leave all other defaults
     and then click `Install`.

3. Verify the operator installations:

    3.1. Switch to the `Operators` → `Installed Operators` page.

    3.2. Make sure the `openshift-logging` project is selected.

    3.3. In the _Status_ column you should see green checks with either
     `InstallSucceeded` or `Copied` and the text _Up to date_.

----

During installation an operator might display a `Failed` status. If the
operator then installs with an `InstallSucceeded` message, you can safely
ignore the `Failed` message. Also, if you're using the `Console` tab, you may
or maynot see the `Status` column. When in doubt, visit the console via the
link.
----

#### Create the Loggging `CustomResource (CR)` instance

Now that we have the operators installed, along with the `CRDs`, we can now
kick off the logging install by creating a Logging `CR`. This will define how
we want to install and configure logging.


1. In the OpenShift Console, switch to the the `Administration` → `Custom Resource Definitions` page.

2. On the `Custom Resource Definitions` page, search for `Logging` in the search field and click `ClusterLogging`.

3. On the `Custom Resource Definition Overview` page, select `Instances` from the `Actions` menu.

4. On the `Cluster Loggings` page, click `Create Cluster Logging`.

5. In the `YAML` editor, replace the code with the following:

https://docs.openshift.com/container-platform/4.11/logging/config/cluster-logging-configuring-cr.html

````
oc apply -f 01-logging.yaml
````

#### Verify the Loggging install

````
oc get pods -n openshift-logging
````

````
oc get daemonset -n openshift-logging
````

````
oc get pvc -n openshift-logging
````
