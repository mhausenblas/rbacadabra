Kubernetes [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) can sometimes appear to be black magic. Let's look behind the curtain.

## Snippets

### Re-use roles across namespaces

Define a cluster role, for example, to view `services`, using:

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


## Commands

### Checking for permissions

Can the service account `dummy` create services in the namespace `danger`?

```sh
$ kubectl auth can-i \
          create services \
          --as=system:serviceaccount:danger:dummy \
          --namespace=danger
```