# Security Context Constraint (SCCs)

Red Hat OpenShift provides security context constraints (SCCs), a security mechanism that restricts access to resources, but not to operations in OpenShift.

SCCs limit the access from a running pod in OpenShift to the host environment. SCCs control:

* Running privileged containers.
* Requesting extra capabilities for a container
* Using host directories as volumes.
* Changing the SELinux context of a container.
* Changing the user ID.

Some containers developed by the community might require relaxed security context constraints to access resources that are forbidden by default, such as file systems, sockets, or to access a SELinux context.

You can run the following command as a cluster administrator to list the SCCs defined by OpenShift:

````bash
oc get scc
````

Create the authorization-scc project.

````bash
oc new-project authorization-scc
````

Deploy an application named gitlab using the container image located at quay.io/redhattraining/gitlab-ce:8.4.3-ce.0. 

This image is a copy of the container image available at docker.io/gitlab/gitlab-ce:8.4.3-ce.0. Verify that the pod fails because the container image needs root privileges.

Deploy the gitlab application.

````bash
oc new-app --name gitlab quay.io/redhattraining/gitlab-ce:8.4.3-ce.0
````

Determine if the application is successfully deployed. It should give an error because this image needs root privileges to properly deploy.

````bash
oc get pods
````

Review the application logs to confirm that the failure is caused by insufficient privileges.

````bash
oc logs pod/gitlab-7d67db7875-gcsjl -f
````

Check if using a different SCC can resolve the permissions problem.

````bash
oc get pod/gitlab-6d7895868d-mnr9r -o yaml | oc adm policy scc-subject-review -f -
````

Create a new service account and assign the anyuid SCC to it.

Create a service account named gitlab-sa.

````bash
oc create sa gitlab-sa
````

Assign the anyuid SCC to the gitlab-sa service account.

````bash
oc adm policy add-scc-to-user anyuid -z gitlab-sa
````

Modify the gitlab application so that it uses the newly created service account. Verify that the new deployment succeeds.

Assign the gitlab-sa service account to the gitlab deployment.

````bash
oc set serviceaccount deployment/gitlab gitlab-sa
````

Verify that the gitlab redeployment succeeds. You might need to run the oc get pods command multiple times until you see a running application pod.

````bash
oc get pods
````

Verify that the gitlab application works properly.

Expose the gitlab application. Because the gitlab service listens on ports 22, 80, and 443, you must use the --port option.

````bash
oc expose service/gitlab --port 80
````

Get the exposed route.

````bash
oc get route
````

## Clean-up

````bash
oc delete project authorization-scc
````

## Links
* [Managing security context constraints](https://docs.openshift.com/container-platform/4.11/authentication/managing-security-context-constraints.html)
