Kubernetes [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) can sometimes appear to be some sort of black magic. So, let's look behind the curtain.

## Snippets

We all love YAML, right? Here's a collection of annotated RBAC-related YAML snippets you can build on.

### Reuse roles

To re-use a role across namespaces, first define a cluster role, for example, to view `services`, using:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: perm-svc-view
rules:
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - get
  - list
```

Assign above cluster role to user `greta`, allowing her to view `services` in the namespace `dev123` using:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: assign-perm-greta-svc-view
  namespace: dev123
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: perm-svc-view
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: greta
```

!!! tip
    To validate if the permissions have been granted, have a look at [checking for permissions](#checking-for-permissions).

### Give namespace admin

### Allow developer to deploy


## Commands

Some `kubectl` commands that might come in handy.

### Creating roles

To create a cluster role:

```sh
kubectl create clusterrole perm-view-deploys \
        --verb=get --verb=list \
        --resource=deployments
```

To create a role in namespace `somens`:

```sh
kubectl create role perm-view-deploys-in-somens \
        --verb=get,list,watch \
        --resource=deployments \
        --namespace=somens
```

### Creating role bindings

To create a cluster role binding, that is, assign a cluster role to user:

```sh
kubectl create clusterrolebinding assign-perm-view-deploys\
        --clusterrole=perm-view-deploys \
        --user=greta
```

To create a role binding, that is, assign role to user in namespace `somens`:

```sh
kubectl create rolebinding assign-perm-view-deploys-in-somens \
        --role=perm-view-deploys-in-somens \
        --user=greta \
        --namespace=somens
```

!!! tip
    If you just want to see what the YAML manifest would look like that the `kuebctl` command creates, that is, no create the resource, append `-o yaml --dry-run` to the respective command.

### Checking for permissions

Can the user `greta` list services in the namespace `dev123`?

```sh
$ kubectl auth can-i \
          list services \
          --as=greta \
          --namespace=dev123
```

Can the service account `dummy` create services in the namespace `danger`?

```sh
$ kubectl auth can-i \
          create services \
          --as=system:serviceaccount:danger:dummy \
          --namespace=danger
```

### Aggregated ClusterRoles

ClusterRoles can be created by combining other clusterroles using aggregationRule. Kubernetes controller manager will take care of filling the rules based on the labels.

To test it out , you can create the below rule in your cluster.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: namespace-admin
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac.example.com/aggregate-to-namespace-admin: "true"
rules: [] # Rules are automatically filled in by the controller manager.
```

``` sh
kubectl create -f ~/namespace-admin.yaml

clusterrole.rbac.authorization.k8s.io/namespace-admin created

```

If you look at the rules section in the below output, you can see that it has no rules.

```sh

 kubectl describe clusterrole namespace-admin
Name:         namespace-admin
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----

```

Let us create another cluster role with the same label

```yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    rbac.example.com/aggregate-to-namespace-admin: "true"
  name: aggregate-to-namespace-admin
rules:
- apiGroups:
  - apps
  resources:
  - deployments 
  verbs:
  - create
  - delete
  - patch
  - update

```

```sh
kubectl create -f ~/aggregate-to-namespace-admin.yaml

clusterrole.rbac.authorization.k8s.io/aggregate-to-namespace-admin created
```
Now check the permissions for namespace-admin role. You can see that controller manager aggregated all the rules based on the labels.

```sh

kubectl describe clusterrole namespace-admin
Name:         namespace-admin
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources         Non-Resource URLs  Resource Names  Verbs
  ---------         -----------------  --------------  -----
  deployments.apps  []                 []              [create delete patch update]
```
 
