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