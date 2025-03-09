---
title: Kubernetes Reference
weight: 40
---

<!-- Variable -->
{{< note title="Operator Install Plan" >}}
```bash
# Find which IP need approval
oc get ip -n my-project -o=jsonpath='{.items[?(@.spec.approved==false)].metadata.name}'

# Batch approve all IPs
oc patch installplan $(oc get ip -n my-project-o=jsonpath='{.items[?(@.spec.approved==false)].metadata.name}') -n my-project --type merge --patch '{"spec":{"approved":true}}'  
```
{{< /note >}}

<!-- RBAC -->
{{< note title="RBAC" >}}
A ClusterRole|Role defines a set of permissions and where it is available, in the whole cluster or just a single Namespace.

A ClusterRoleBinding|RoleBinding connects a set of permissions with an account and defines where it is applied, in the whole cluster or just a single Namespace.

Because of this there are 4 different RBAC combinations and 3 valid ones:

Role + RoleBinding (available in single Namespace, applied in single Namespace)  
ClusterRole + ClusterRoleBinding (available cluster-wide, applied cluster-wide)  
ClusterRole + RoleBinding (available cluster-wide, applied in single Namespace)  
Role + ClusterRoleBinding (NOT POSSIBLE: available in single Namespace, applied cluster-wide)
{{< /note >}}

<!-- Etcd Snapshot & Restore -->
{{< note title="Etcd Snapshot & Restore" >}}
```bash
export ETCDCTL_API=3
etcdctl snapshot save --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=127.0.0.1:2379 /opt/etcd-backup.db

etcdctl --data-dir=/root/etcd-from-backup --endpoints=https://172.0.0.8:2379 \
  --cert=/root/kubernetes/pki/etcd/server.crt \
  --key=/root/kubernetes/pki/etcd/server.key \
  --cacert=/root/kubernetes/pki/etcd/ca.crt \
  snapshot restore /opt/etcd-backup.db
```
{{< /note >}}

