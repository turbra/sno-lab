# sno-lab
## Single Node OpenShift - Home lab

### Deploy nfs external provisioner using the [nfs-subdir-external-provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)

### Create local user and grant cluster admin privileges

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
