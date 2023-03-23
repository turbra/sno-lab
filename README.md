# sno-lab
## [Single Node OpenShift](https://docs.openshift.com/container-platform/4.12///installing/installing_sno/install-sno-preparing-to-install-sno.html) - Home lab

## This will cover the following:
- How to deploy nfs external provisioner using the [nfs-subdir-external-provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)
- How to create local user and grant cluster admin privileges

#### Create new project
`oc new-project nfs-auto`

#### Set the subject of the RBAC objects to the current namespace where the provisioner is being deployed
```bash
NAMESPACE=`oc project -q`
sed -i'' "s/namespace:.*/namespace: $NAMESPACE/g" rbac.yaml deployment.yaml
oc create -f rbac.yaml
oc adm policy add-scc-to-user hostmount-anyuid system:serviceaccount:$NAMESPACE:nfs-client-provisioner
```
#### Deploy NFS Client Provisioner
`oc create -f deployment.yaml`

#### Deploy Storage Class
`oc create -f class.yaml`

#### Test it all
`oc create -f test-claim.yaml -f test-pod.yaml`

#### Define default storage class
`oc patch storageclass nfs-client -p '{"metadata": {"annotations": {"storageclass.kubernetes.io/is-default-class": "true"}}}'`


# [Creating an HTPasswd Identity Provider](https://docs.openshift.com/container-platform/4.12/authentication/identity_providers/configuring-htpasswd-identity-provider.html)

#### Use the htpasswd command to add users and passwords to the file we just created
```bash
htpasswd -c -B -b htpasswd user pass123
```

#### Create a secret in the `openshift-config` project
`oc create secret generic htpass-secret --from-file=htpasswd -n openshift-config`

#### Export the existing OAUTH resource to a file, for us to modify
`oc get oauth cluster -o yaml > auth-provider.yaml`

#### Edit the auth-provider.yaml with our new identity provider
```yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: my_htpasswd_provider 
    mappingMethod: claim 
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-secret 
```

#### Apply the custom resource defined in the previous step
`oc replace -f auth-provider.yaml`

#### Assign cluster-admin role to a user
`oc adm policy add-cluster-role-to-user cluster-admin <USER>`

### Add users to the IDP
#### Extract the file data from the secret
`oc extract secret/localusers -n openshift-config --to /home/user/ --confirm`

#### Add an entry to the htpasswd file for the additional user
`htpasswd -b /home/user/htpasswd user99 password`

#### Update the secret after adding additional users
`oc set data secret/localusers --from-file htpasswd=/home/user/htpasswd -n openshift-config`
