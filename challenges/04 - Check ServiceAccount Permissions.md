# How to QUICKLY check whether an action is allowed through a ServiceAccount

### can-i
`can-i` simply checks with the API to see if an action can be performed. It can take the following options 
```bash
kubectl auth can-i VERB [TYPE | TYPE/NAME | NONRESOURCEURL]

kubectl auth can-i delete pod/compute 
```

### --as
By default `can-i` checks if the current user has permission to perform an action. To check it for a specific `user` we use `--as`

We can use the `--as` with or without `auth can-i`
```bash
kubectl auth can-i VERB [TYPE | TYPE/NAME | NONRESOURCEURL] --as [USERNAME]

kubectl auth can-i delete pod/compute --as sid
```

### Service Accounts
We can use the same `--as` with or without `auth can-i` to see what actions a `serviceaccount` can perform before using that in a `pod`.

```bash
alias k=kubectl

kubectl create ns ks001uv
kubectl get sa -n ks001uv

NAME      SECRETS   AGE
default   1         66m
```

The default `serviceaccount` in a `namespace` has no permissions other than those of an unauthenticated user.

Let's try to list the `services` in `ks001uv` namespace using the `default` serviceaccount
```bash
kubectl auth can-i get svc --as=system:serviceaccount:<namespace>:<service-account-name>

#with auth can-i
kubectl auth can-i get svc --as=system:serviceaccount:ks001uv:default
no

# with out auth can-i
kubectl get svc --as=system:serviceaccount:ks001uv:default
Error from server (Forbidden): services is forbidden: User "system:serviceaccount:ks001uv:default" cannot list resource "services" in API group "" inthe namespace "ks001uv"

```
![no](https://user-images.githubusercontent.com/28925814/100622000-5adb9780-3346-11eb-98cb-a5d8a299f6f7.png)
![no-as](https://user-images.githubusercontent.com/28925814/100620885-f5d37200-3344-11eb-8fe2-3c8b2081336b.png)

**We should get `no` or `Forbidden` response as the `default` serviceaccount cannot `list` the services.**

### Role and Rolebinding
Let us create a `Role` and `Rolebinding` which would allow the `default` serviceaccount in `ks001uv` namespace to `list` the `services`.

```yaml
#### saving this to /root/role-role-binding.yaml
### Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: null
  name: list-svc-role
  namespace: ks001uv
rules:
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - get
  - list
---
### Role Binding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: null
  name: list-svc-role-binding
  namespace: ks001uv
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: list-svc-role
subjects:
- kind: ServiceAccount
  name: default
  namespace: ks001uv
```
```bash
kubectl apply -f /root/role-role-binding.yaml

role.rbac.authorization.k8s.io/list-svc-role created
rolebinding.rbac.authorization.k8s.io/list-svc-role-binding created
```

### Testing Service Account
```bash
#with auth can-i
kubectl auth can-i get svc --as=system:serviceaccount:ks001uv:default
yes

# with out auth can-i
kubectl get svc --as=system:serviceaccount:ks001uv:default
<!-- gives the list of services>
```

Often times we create a `serviceaccount` create/assign new `role` to it and then use them in a `pod` specification. Before using it in a `pod` spec, we can quickly check the `permissions` assigned to the serviceaccount using this method.