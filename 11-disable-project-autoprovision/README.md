## Disabling Project Self-Provisioning

OpenShift, by default, allows authenticated users to create *Projects* to
logically house their applications. This feature enables admins to provide a
"self service" feature that was popularized by the advent of the "PaaS"
(Platform as a Service) model.

This feature may not be suitable for all situations, and many administrators
may want to have more control over Projects and who (if anyone) can create
them. Some of these use cases may be:

### Examine the Clusterrolebinding
In order to examine the `clusterrolebinding` for `self-provisioners`; you
need to be *kubeadmin*.

View the `self-provisioners` cluster role binding usage by running the `oc describe` command:

````
oc describe clusterrolebinding.rbac self-provisioners
````


Here, it's saying that the group `system:authenticated:oauth` (which is a
group that every user that has authenticated is a part of, by default), is
bound to the `self-provisioners` role. This role is a built-in role in
OpenShift that allows the users to create projects.

To view the configuration itself, inspect the yaml by running the following:

````
oc get clusterrolebinding.rbac self-provisioners -o yaml
````

#### Removing Self Provisioning Projects
To remove the `self-provisioner` cluster role from the group
`system:authenticated:oauth` you need to remove that group from the role
binding.

````
oc patch clusterrolebinding.rbac self-provisioners -p '{"subjects": null}'
````

Automatic updates reset the cluster roles to a default state. In order to
disable this, you need to set the annotation
`rbac.authorization.kubernetes.io/autoupdate` to `false` by running:

````
oc patch clusterrolebinding.rbac self-provisioners -p '{ "metadata": { "annotations": { "rbac.authorization.kubernetes.io/autoupdate": "false" } } }'
````

View the new configuration:

````
oc get clusterrolebinding.rbac self-provisioners -o yaml
````

It should now have no `subjects` in the YAML.
