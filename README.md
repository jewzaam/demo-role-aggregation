# Demo of Role Aggregation

I wanted to share how role aggregation works in Kubernetes and needed a simple example.  This capability is used _everywhere_ and can be hard to unwind how it is used from what it is doing if looking at a life cluster.

## What is Role Aggregation and how does it work?

https://kubernetes.io/docs/reference/access-authn-authz/rbac/#aggregated-clusterroles

In short, it is a way to take the rules from multiple `ClusterRoles` and pull them into another `ClusterRole`.

It's all based on labels on `ClusterRoles`.  You have an "aggregating" `ClusterRole` in which you define the aggregation rule.  When a `ClusterRole` has labels that match the aggregation rules on the "aggregating" `ClusterRole` all the `rules` from that matched `ClusterRole` are merged into the "aggregating" `ClusterRole`.

NOTE: any `rules` in the "aggregating" `ClusterRole` are lost.

## Demo: Setup

This repo contains 3 `ClusterRoles`:

- nmalik-clusterrole
    - all verbs for `ClusterRoles` and `ClusterRoleBindings`
    - has label `nmalik/aggregate-to-parent` with value `"true"`
- nmalik-oauth
    - all verbs for `OAuthClientAuthorizations`
    - has label `nmalik/aggregate-to-parent` with value `"true"`
- nmalik-parent
    - no rules, would be clobbered by aggregation
    - has aggregation rule to match where label `nmalik/aggregate-to-parent` with value `"true"`

In the demo, `ClusterRole` nmalik-parent is the "aggregating" `ClusterRole`.

## Demo: Execute


### Create empty nmalik-parent

Create the nmalik-parent `ClusterRole`.

```bash
$ kubectl create -f manifests/00_nmalik-parent.clusterrole.yaml 
clusterrole.rbac.authorization.k8s.io/nmalik-parent created
```

Observe that this does not have any rules initially.

```bash
$ kubectl get clusterrole nmalik-parent -o yaml
aggregationRule:
  clusterRoleSelectors:
  - matchExpressions:
    - key: nmalik/aggregate-to-parent
      operator: In
      values:
      - "true"
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: "2020-03-24T14:18:27Z"
  name: nmalik-parent
  resourceVersion: "96833"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterroles/nmalik-parent
  uid: 456b7178-48c0-417d-abf5-b4010f859496
rules: null
```

### Create nmalik-clusterrole

Create the nmalik-clusterrole `ClusterRole`.

```bash
$ kubectl create -f manifests/10_nmalik-clusterrole.clusterrole.yaml 
clusterrole.rbac.authorization.k8s.io/nmalik-clusterrole created
```

Observe that the nmalik-parent `ClusterRole` rules are updated even though we did not change it directly.

```bash
$ kubectl get clusterrole nmalik-parent -o yaml
aggregationRule:
  clusterRoleSelectors:
  - matchExpressions:
    - key: nmalik/aggregate-to-parent
      operator: In
      values:
      - "true"
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: "2020-03-24T14:18:27Z"
  name: nmalik-parent
  resourceVersion: "97578"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterroles/nmalik-parent
  uid: 456b7178-48c0-417d-abf5-b4010f859496
rules:
- apiGroups:
  - rbac.authorization.k8s.io
  resources:
  - clusterroles
  - clusterrolebindings
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
```

### Create nmalik-oauth

Create the nmalik-oauth `ClusterRole`.

```bash
$ kubectl create -f manifests/20_nmalik-oauth.clusterrole.yaml 
clusterrole.rbac.authorization.k8s.io/nmalik-oauth created
```

Observe that the nmalik-parent `ClusterRole` rules are updated even though we did not change it directly.

```bash
$ kubectl get clusterrole nmalik-parent -o yaml
aggregationRule:
  clusterRoleSelectors:
  - matchExpressions:
    - key: nmalik/aggregate-to-parent
      operator: In
      values:
      - "true"
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: "2020-03-24T14:18:27Z"
  name: nmalik-parent
  resourceVersion: "98193"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterroles/nmalik-parent
  uid: 456b7178-48c0-417d-abf5-b4010f859496
rules:
- apiGroups:
  - rbac.authorization.k8s.io
  resources:
  - clusterroles
  - clusterrolebindings
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
- apiGroups:
  - ""
  - oauth.openshift.io
  resources:
  - oauthclientauthorizations
  verbs:
  - delete
  - get
  - list
  - watch
```

## Delete nmalik-clusterrole

Delete the nmalik-clusterrole `ClusterRole`.

```bash
$ kubectl delete -f manifests/10_nmalik-clusterrole.clusterrole.yaml 
clusterrole.rbac.authorization.k8s.io "nmalik-clusterrole" deleted
```

Observe that the nmalik-parent `ClusterRole` rules are updated even though we did not change it directly.  The rules from nmalik-clusterrole are removed!

```bash
$ kubectl get clusterrole nmalik-parent -o yaml
aggregationRule:
  clusterRoleSelectors:
  - matchExpressions:
    - key: nmalik/aggregate-to-parent
      operator: In
      values:
      - "true"
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: "2020-03-24T14:18:27Z"
  name: nmalik-parent
  resourceVersion: "98739"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterroles/nmalik-parent
  uid: 456b7178-48c0-417d-abf5-b4010f859496
rules:
- apiGroups:
  - ""
  - oauth.openshift.io
  resources:
  - oauthclientauthorizations
  verbs:
  - delete
  - get
  - list
  - watch
```
