# Configure htpasswd

1. Add an entry for two htpasswd users, admin and developer. Assign admin a password of redhat and developer a password of developer.

    1.1. Create an HTPasswd authentication file named htpasswd. Add the admin user with the password of redhat. The name of the file is arbitrary, but this exercise use the htpasswd file.

    Use the htpasswd command to populate the HTPasswd authentication file with the user names and encrypted passwords. The -B option uses bcrypt encryption. By default, the htpasswd command uses MD5 encryption when you do not specify an encryption option.

    Execute the following:

    ````bash
    htpasswd -c -B -b htpasswd admin redhat
    ````

    1.2. Add the developer user with a password of developer to the htpasswd file.

    ````bash
    htpasswd -b htpasswd developer developer
    ````

    1.3. Review the contents of the /htpasswd file and verify that it includes two entries with hashed passwords: one for the admin user and another for the developer user.

    ````bash
    cat htpasswd
    ````

2. Create a secret that contains the HTPasswd users file.

    2.1. Create a secret from the /htpasswd file. To use the HTPasswd identity provider, you must define a secret with a key named htpasswd that contains the HTPasswd user file /htpasswd.

    ````bash
    oc create secret generic localusers --from-file htpasswd=htpasswd -n openshift-config
    ````

    2.2. Assign the admin user the cluster-admin role.

    ````bash
    oc adm policy add-cluster-role-to-user cluster-admin admin
    ````

3. Update the HTPasswd identity provider for the cluster so that your users can authenticate. Configure the custom resource file and update the cluster.

    3.1. Apply the custom resource defined in the previous step.

    ````bash
    oc replace -f 01-oauth.yaml
    ````

# RBAC lab

## Developer as cluster reader

````bash
oc adm policy add-cluster-role-to-user cluster-reader developer
````

Check the permissions

Deletting the assigned role

````bash
oc adm policy remove-cluster-role-from-user cluster-reader developer
````

## As project admin

````bash
oc adm policy add-role-to-user admin developer -n application-basics-cli
````

Check the permissions

Deletting the assigned role

````bash
oc adm policy remove-role-from-user admin developer -n application-basics-cli
````

## Developer as cluster reader via yaml

````bash
oc create -f 02-developer-cluster-reader.yaml
````
