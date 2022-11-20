## Project Request Template, Quotas, Limits

Out of the box, OpenShift does not restrict the quantity of objects or amount of
resources (eg: CPU, memory) that they can consume within a *Project*. Further,
users may create an unlimited number of *Projects*. In a world with no
constraints, or in a POC-type environment, that would be fine. But reality calls
for a little restraint.

### Background: Project Request Template
When users invoke the `oc new-project` command, a new project request flow is
initiated. During part of this this workflow, a default project *Template* is
processed to create the newly requested project.

#### View the Default Project Request Template
To view the built-in default *Project Request Template*, execute the
following command:

````
oc adm create-bootstrap-project-template -o yaml
````

#### Modify the Project Request Template

````
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: ${PROJECT_NAME}-quota <1>
  spec:
    hard:
      pods: 10
      requests.cpu: 4000m
      requests.memory: 8Gi
      requests.storage: 50Gi
      persistentvolumeclaims: 5
````

````
- apiVersion: v1
  kind: LimitRange
  metadata:
    name: ${PROJECT_NAME}-limits
    creationTimestamp: null
  spec:
    limits:
      -
        type: Container
        max:
          cpu: 4000m
          memory: 1024Mi
        min:
          cpu: 10m
          memory: 5Mi
        default:
          cpu: 4000m
          memory: 1024Mi
        defaultRequest:
          cpu: 100m
          memory: 512Mi
````

### Installing the Project Request Template
With this background in place, let's go ahead and actually tell OpenShift to
use this new *Project Request Template*.

#### Create the Template
As we discussed earlier, a *Template* is just another type of OpenShift object.
The `oc` command provides a `create` function that will take YAML/JSON as input
and simply instantiate the objects provided.

Go ahead and execute the following:

````
oc create -f 01-template.yaml -n openshift-config
````

````
oc get template -n openshift-config
````

#### Setting the Default ProjectRequestTemplate
The default *projectRequestTemplate* is part of the OpenShift API Server
configuration. This configuration is ultimately stored in a *ConfigMap* in
the `openshift-apiserver` project. You can view the API Server configuration
with the following command:

````
oc get cm config -n openshift-apiserver -o jsonpath --template="{.data.config\.yaml}" | jq  .projectConfig
````

There is an OpenShift operator that looks at various *CustomResource* (CR)
instances and ensures that the configurations they define are implemented in
the cluster. In other words, the operator is ultimately responsible for
creating/modifying the *ConfigMap*. You can see in the `jq` output that there
is a `projectRequestMessage` but no `projectRequestTemplate` defined. There
is currently no CR specifying anything, so the operator has configured the
cluster with the "stock" settings. To add the default project request
tempalate configuration, a CR needs to be created. The
*CustomResource* will look like:

````
oc edit project.config.openshift.io cluster
````

````
apiVersion: "config.openshift.io/v1"
kind: "Project"
metadata:
  name: "cluster"
  namespace: ""
spec:
  projectRequestTemplate:
    name: "project-request"
````

Notice the *projectRequestTemplate* name matches the name of the template we
created earlier in the `openshift-config` project.

````
sleep 30
oc rollout status deploy apiserver -n openshift-apiserver
````

This can now be verified by viewing the implemented
configuration:

````
oc get cm config -n openshift-apiserver -o jsonpath --template="{.data.config\.yaml}" | jq .projectConfig
````


#### Create a New Project
When creating a new project, you should see that a *Quota* and a *LimitRange*
are created with it. First, create a new project called `template-test`:

````
oc new-project template-test
````

Then, use `describe` to look at some of this *Project's* details:

````
oc describe project template-test
````

````
oc get quota -n template-test
````

````
oc describe quota -n template-test
````

````
oc get limitrange -n template-test
````

````
oc describe limitrange -n template-test
````

### Clean Up

````
oc delete project template-test
oc delete -f support/project_request_template.yaml -n openshift-config
cat <<EOF | oc apply -f -
apiVersion: "config.openshift.io/v1"
kind: "Project"
metadata:
  name: "cluster"
  namespace: ""
spec:
  projectRequestMessage: ""
  projectRequestTemplate:
    name: ""
EOF
sleep 30
oc rollout status deploy apiserver -n openshift-apiserver
````
