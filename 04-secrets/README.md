# Managing Sensitive Information with Secrets

Create the authorization-secrets project.

````bash
oc new-project authorization-secrets
````

Create a secret with the credentials and connection information to access a MySQL database.

````bash
oc create secret generic mysql \
--from-literal user=myuser --from-literal password=redhat123 \
--from-literal database=test_secrets --from-literal hostname=mysql
````

Deploy a database and add the secret for user and database configuration.

````bash
oc create -f 01-mysql-env.yaml -n authorization-secrets
````

Verify that the database now authenticates using the environment variables initialized from the mysql secret.

Open a remote shell session to the mysql pod in the Running state.

````bash
oc rsh mysql-HASH
````

Start a MySQL session to verify that the environment variables initialized by the mysql secret were used to configure the mysql application.

````bash
mysql -u myuser --password=redhat123 test_secrets -e 'show databases;'
````

To demonstrate how a secret can be mounted as a volume, mount the mysql secret to the /run/secrets/mysql directory within the

````bash
oc set volume deployment/mysql --add --type secret \
 --mount-path /run/secrets/mysql --secret-name mysql
````

Open a remote shell session to the mysql pod in the Running state.

````bash
oc rsh mysql-HASH
````

List mount points in the pod containing the mysql pattern. Notice that the mount point is backed by a temporary file system (tmpfs). This is true for all secrets that are mounted as volumes.

````bash
df -h | grep mysql
````

Examine the files mounted at the /run/secrets/mysql mount point. Each file matches a key name in the secret, and the content of each file is the value of the associated key.

````bash
ls /run/secrets/mysql

for FILE in $(ls /run/secrets/mysql)
do
echo "${FILE}: $(cat /run/secrets/mysql/${FILE})"
done
````

Close the remote shell session to continue from your workstation machine.





Create a new application that uses the MySQL database.

Create a new application using the redhattraining/famous-quotes image from Quay.io.

````bash
oc apply -f 03-mysql-service.yaml -n authorization-secrets
oc apply -f 04-quotes-app.yaml -n authorization-secrets
````

 Verify that the quotes pod successfully connects to the database and that the application displays a list of quotes.

Examine the pod logs using the oc logs command. The logs indicate a successful database connection.

````bash
oc logs  -f quotes-8595f8bfcf-96rmn | head -n2
````

Expose the quotes service so that it can be accessed from outside the cluster.

````bash
oc apply -f 05-quotes-service.yaml -n authorization-secrets
oc expose service quotes
````

Verify the application host name.

````bash
oc get route
````

Access the application status REST API entry point to test the database connection.

````bash
curl -s http://quotes-authorization-secrets.apps.cluster-754d.sandbox478.opentlc.com/status
````

 Test application functionality by accessing the random REST API entry point.

````bash
curl -s http://quotes-authorization-secrets.apps.cluster-754d.sandbox478.opentlc.com/random
````

Remove the authorization-secrets project.

````bash
oc delete project authorization-secrets
````
