# Openshift-Study-Guide

## Installation
Install openshift 4+ on local workstation using [crc](https://developers.redhat.com/products/codeready-containers)

Note, your'e given to logins, 'developer' and 'kubeadmin' you can use either one to do `oc login ...`

E.g. 


```
oc login -u kubeadmin -p e4FEb-9dxdF-9N2wH-Dj7B8 https://api.crc.testing:6443
```

or 

```
oc login -u developer -p developer https://api.crc.testing:6443
```

The 'developer' user has less permissions, e.g.:

```
oc login -u developer -p developer https://api.crc.testing:6443
Login successful.
$ kubectl get pods --namespace default
Error from server (Forbidden): pods is forbidden: User "developer" cannot list resource "pods" in API group "" in the namespace "default"
```

However by default, our developer user is attached to the 'sudoer' clusterrole. so can activate extra privileges like this:

```
kubectl get pods --namespace default --as system:admin
```

You can find more info about this sudoer clusterrole:

```
oc get clusterroles -o yaml sudoer
```

Here's how to assign [sudoer role to a user](https://docs.openshift.com/container-platform/4.2/authentication/impersonating-system-admin.html):

```
$ oc create clusterrolebinding <any_valid_name> --clusterrole=sudoer --user=<username>
```


To [get a list of all users](https://docs.okd.io/latest/admin_guide/manage_users.html#managing-users-viewing-user-and-identity-lists):

```
$ oc get user
NAME        UID                                    FULL NAME   IDENTITIES
developer   15493539-0fbe-11ea-94e3-0a580a8000de               htpasswd_provider:developer

$ oc get identity
NAME                          IDP NAME            IDP USER NAME   USER NAME   USER UID
htpasswd_provider:developer   htpasswd_provider   developer       developer   15493539-0fbe-11ea-94e3-0a580a8000de
```

(I think) [a user can only be attached to one role](https://docs.openshift.com/container-platform/3.7/admin_guide/manage_rbac.html#viewing-cluster-bindings):

```
oc get clusterrolebinding.rbac developer -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  creationTimestamp: "2019-10-31T08:06:10Z"
  name: developer
  resourceVersion: "230405"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterrolebindings/developer
  uid: 4c75f31c-fbb5-11e9-9f35-525400d602f2
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: sudoer
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: developer
```

This says we have a cluster-rolebinding called 'developer', this crb attaches the user 'developer' to the clusterrole 'sudoer'. 


Instead of doing (while logged in as 'developer' user):

```
kubectl get pods --namespace default --as system:admin
```

you can instead assign the 'view' clusterrole to the 'developer' user for the 'default' namespace:

```
oc adm policy add-role-to-user view developer -n default
```

Instead of the view role, you can use the 'edit' role, if you want to give user write privileges to the namespace. or 'admin' if you want to make the user joint owner of the namespace. 


You check who you're logged in as:

```
oc whoami
```

You can open openshift web console:

```
crc console
```

Create new project:

```
oc new-project my-new-project
```

A openshift [project](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.2/html-single/applications/index#working-with-projects) is a kubernetes namespace with extra annotations, e.g.:

```
$ oc new-project hello-openshift \
    --description="This is an example project" \
    --display-name="Hello OpenShift"
```



