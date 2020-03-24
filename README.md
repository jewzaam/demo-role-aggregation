# Role Aggregation

[//]: <> (TOC generated from https://ecotrust-canada.github.io/markdown-toc/)
[//]: <> (making a hidden comment from https://stackoverflow.com/questions/4823468/comments-in-markdown)

- [Role Aggregation](#role-aggregation)
  * [What is it and how does it work?](#what-is-it-and-how-does-it-work-)
- [Demonstration](#demonstration)
  * [Overview](#overview)
  * [1. Create empty nmalik-parent](#1-create-empty-nmalik-parent)
  * [2. Create nmalik-clusterrole](#2-create-nmalik-clusterrole)
  * [3. Create nmalik-oauth](#3-create-nmalik-oauth)
  * [4. Delete nmalik-clusterrole](#4-delete-nmalik-clusterrole)
- [Additional Information](#additional-information)
  * [Label Key / Value](#label-key---value)
  * [Rules on the "aggregating" ClusterRole](#rules-on-the--aggregating--clusterrole)

I wanted to share how role aggregation works in Kubernetes and needed a simple example.  This capability is used _everywhere_ and can be hard to unwind how it is used from what it is doing if looking at a life cluster.

## What is it and how does it work?

https://kubernetes.io/docs/reference/access-authn-authz/rbac/#aggregated-clusterroles

In short, it is a way to take the rules from multiple `ClusterRoles` and pull them into another `ClusterRole`.

It's all based on labels on `ClusterRoles`.  You have an "aggregating" `ClusterRole` in which you define the aggregation rule.  When a `ClusterRole` has labels that match the aggregation rules on the "aggregating" `ClusterRole` all the `rules` from that matched `ClusterRole` are merged into the "aggregating" `ClusterRole`.

NOTE: any `rules` in the "aggregating" `ClusterRole` are lost.

# Demonstration

## Overview

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


## 1. Create empty nmalik-parent

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

## 2. Create nmalik-clusterrole

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

## 3. Create nmalik-oauth

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

## 4. Delete nmalik-clusterrole

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
# Additional Information

Just some other info to keep in mind.

## Label Key / Value

The label key and value does not matter, it can be any values.  The defacto standard for a the key appears to be `"<prefix>/aggregate-to-<role>"` with values being either `"true"` or some value that provides context.

## Rules on the "aggregating" ClusterRole

Any rules on the "aggregating" `ClusterRole` (the one that has the aggregation rule) will be thrown away.  Simply do not set any rules.  You can set it with `rules: null` or `rules: []`.  See [manifests/00_nmalik-parent.clusterrole.yaml](manifests/00_nmalik-parent.clusterrole.yaml) for the example in this repo.