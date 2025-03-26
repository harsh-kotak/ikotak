---
title: "Kubernetes Interview Questions"
date: 2025-03-08T06:15:40+06:00
hero: ovs.svg
description: Kubernetes Interview QUestions
menu:
  sidebar:
    name: Kubernetes Prep
    identifier: kube-prep
    weight: 30
---

# Kubernetes Interview Prep Guide

This page serves as a comprehensive technical guide for Kubernetes interviews. Topics are grouped and answered in-depth to help with preparation for DevOps, SRE, and Platform Engineer roles.

Each section includes detailed technical Q&A, YAML examples, comparisons, and real-world use cases.

---

## Linux and Container Fundamentals

#### Q1: <kbd>What are Linux namespaces?</kbd>
- Namespaces provide process isolation in Linux.
- Types include:
  - `pid`: process IDs
  - `net`: networking
  - `mnt`: mount points
  - `uts`: hostname
  - `ipc`: inter-process communication
  - `user`: UID/GID
- Each container typically runs in its own set of namespaces to isolate processes from others.


#### Q2: <kbd>What are cgroups and how do they work?</kbd>
- Control Groups (cgroups) limit and monitor resource usage per process or container.
- Manage CPU, memory, I/O, and network bandwidth.
- Kubernetes uses cgroups (via the container runtime) to enforce pod resource requests/limits.


#### Q3: <kbd>What does it mean when a container is a "tar of tar"?</kbd>
- Container images are built in layers using a union filesystem.
- Each layer is a tarball of the diff from the previous layer.
- The final image is a "tar of tars" that gets extracted when the container is started.
- Tools like `ctr`, `crictl`, or `docker` handle pulling and extracting these tarballs.


#### Q4: <kbd>How do containers relate to Kubernetes?</kbd>
- Kubernetes schedules and manages containers using Pods.
- It uses a container runtime (e.g., containerd) to start/stop containers.
- Kubernetes adds orchestration features (health checks, scaling, service discovery, etc.) on top of container runtimes.


---

## Core Kubernetes Concepts

#### Q5: <kbd>How does Kubernetes perform service discovery?</kbd>
Kubernetes provides two main methods for service discovery:

1. **Environment Variables**: Injected into pods at creation for each service in the same namespace.
2. **DNS-based discovery (CoreDNS)**:
   - Services are resolvable using DNS like `<service>.<namespace>.svc.cluster.local`.
   - DNS records are automatically updated when services scale or IPs change.
   - Supports `A` records for ClusterIP and `SRV` records for ports.


#### Q6: <kbd>What happens when you run `kubectl apply -f file.yaml`?</kbd>
- Generates a **merge patch** based on the difference between the last-applied state and the new manifest.
- Sends this to the API server.
- The controller reconciles the new desired state.


#### Q7: <kbd>Explain the Kubernetes Control Plane components.</kbd>
- **kube-apiserver**: Main entrypoint; handles authentication, validation, and API requests.
- **etcd**: Distributed key-value store holding cluster state.
- **kube-scheduler**: Assigns pods to nodes based on constraints and priorities.
- **kube-controller-manager**: Manages controllers (e.g., replicaset, endpoints).
- **cloud-controller-manager**: Interfaces with cloud-specific APIs for load balancers, routes, volumes, etc.

Interaction: User → kubectl → API Server → etcd + controllers → kubelet (on node) → container runtime


#### Q8: <kbd>What are taints and tolerations?</kbd>
- **Taints** repel pods from being scheduled on nodes unless pods explicitly tolerate them.
- **Tolerations** let pods match and schedule onto tainted nodes.


#### Q9: <kbd>What's the difference between readiness and liveness probes?</kbd>
- **Readiness Probe**: Signals when a pod is ready to receive traffic. If it fails, the Pod is removed from Service endpoints.
- **Liveness Probe**: Signals if the container is alive; failing triggers a restart. Prevents stuck or deadlocked apps from hanging.
- Probe types: `httpGet`, `exec`, `tcpSocket`, and `grpc`


---

## Workload Management

#### Q10: <kbd>How do rolling updates and rollbacks work?</kbd>
- Managed by the **Deployment Controller**.
- Controlled by `maxSurge` and `maxUnavailable`.
- Kubernetes creates new replicas while phasing out old ones.
- Supports rollback using `kubectl rollout undo deployment <name>`.


#### Q11: <kbd>What are init containers and how are they different from app containers?</kbd>
- Run before any app containers start.
- Run sequentially, each must succeed before the next starts.
- Used for setup tasks (e.g., pulling secrets, waiting for services).

Differences:
- Cannot run in parallel.
- Can have different images/tools than main containers.
- Once complete, they are not restarted.

Example use-case:
- Init container downloads configuration from a remote server.
- App container starts only after config is ready.


#### Q12: <kbd>What is a sidecar container?</kbd>
- A sidecar runs in the same pod as the main app.
- Shares network, volumes, and lifecycle.
- Common use cases:
  - Service mesh proxy (e.g., Envoy in Istio)
  - Logging agents (e.g., Fluent Bit)
  - Monitoring and tracing (e.g., OpenTelemetry collector)


#### Q13: <kbd>What are StatefulSets and how are they different from Deployments?</kbd>
- Manages **stateful** apps.
- Pods have stable hostnames and persistent storage.
- Ordered startup, scaling, and termination.


#### Q14: <kbd>How does the Horizontal Pod Autoscaler (HPA) work?</kbd>
- Automatically scales pods based on metrics.
- Commonly scales on CPU or memory.
- Requires the Metrics Server.
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
```


---

## Storage

#### Q15: <kbd>How do Persistent Volumes and Claims work?</kbd>
- **PV**: Cluster-provisioned storage (manually or dynamically).
- **PVC**: Pod-level request for storage.
- **StorageClass**: Defines provisioners like AWS EBS, GCE PD, or Ceph.


#### Q16: <kbd>What are the reclaim policies?</kbd>
- `Retain`: Manual intervention needed.
- `Delete`: Deletes storage upon claim deletion.
- `Recycle`: Deprecated.


---

## Network

#### Q17: <kbd>How does network communication work between Pods?</kbd>
Kubernetes enforces the flat network model:
- Every Pod gets a unique IP.
- Pods can communicate with each other without NAT.

Network plugins (CNI) provide this functionality:
- Examples: Calico, Flannel, Cilium, Weave.
- CNI handles: IP allocation, Network routing, Firewalling (optional)

Pod-to-Pod communication:
- Routed through the CNI.
- On the same node: uses virtual Ethernet (veth) pairs.
- Across nodes: routed via overlay network or BGP (Calico).


#### Q18: <kbd>How do Network Policies work in Kubernetes?</kbd>
Network Policies control traffic flow at the IP and port level.
- Deny-all by default (only if a policy is applied).
- You need a CNI plugin that supports Network Policies (e.g., Calico, Cilium).

Types of rules:
- Ingress: Controls incoming traffic to Pods.
- Egress: Controls outbound traffic from Pods.

Match criteria:
- podSelector: Select target pods.
- namespaceSelector: Match source/destination namespaces.
- ipBlock: Match IP ranges (e.g., 10.0.0.0/8).

Example: Allow only frontend to talk to backend on port 80
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      role: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 80
```


---

## Security and Access Control

#### Q19: <kbd>What is RBAC in Kubernetes?</kbd>
- Controls API access with:
  - **Role / ClusterRole**: Defines permissions.
  - **RoleBinding / ClusterRoleBinding**: Assigns them to users or service accounts.


#### Q20: <kbd>How are ServiceAccounts and secrets used?</kbd>
- ServiceAccount: Identity for processes inside pods to authenticate to the API Server.
- Kubernetes automatically mounts a token inside pods at `/var/run/secrets/kubernetes.io/serviceaccount/token`.
- Secrets: Store sensitive data like API keys, certs, passwords.
  - Base64 encoded (not encrypted by default).
  - Can be mounted as volumes or injected as env vars.

Improving security:
- Use RBAC to restrict service accounts.
- Enable BoundServiceAccountTokenVolume for token expiration and rotation.
- Use KMS providers (Vault) for secret encryption at rest.


#### Q21: <kbd>What are PodSecurity Standards (PSS)?</kbd>
- Replaces PodSecurityPolicy (PSP): Deprecated in Kubernetes 1.21, removed in 1.25.
- PSS Enforce security rules at the namespace level.
- Modes: `enforce`, `warn`, `audit`
- Levels:
  - `privileged`: No restrictions
  - `baseline`: Common restrictions
  - `restricted`: Strictest settings

#### Q22: <kbd>What are OpenShift SCCs and how do they work?</kbd>
OpenShift uses Security Context Constraints (SCCs) to control what Pods can do — they are more powerful and centralized than standard Kubernetes PodSecurity policies.
- SCCs define constraints on pod security:
  - UID ranges, SELinux context, allowed volumes, capabilities.
  - Applied cluster-wide based on service accounts or user groups.

```bash
# Running a container as root will fail in OpenShift unless the pod is associated with the anyuid SCC
oc adm policy add-scc-to-user anyuid -z my-sa
```


#### Q23: <kbd>How do you ensure secure communications between services in Kubernetes?</kbd>
1. **Network Policies:**
  - Limit which Pods can talk to which.
  - Enforce east-west traffic control.

2. **TLS/mTLS:**
  - Use service mesh (e.g., Istio) for automatic mTLS.
  - Alternatively, apps can implement TLS themselves.

3. **API Access Control:**
  - Use RBAC + ServiceAccounts.
  - Use NetworkPolicy and Ingress rules to protect APIs.

4. **Pod Security:**
  - Use PodSecurity Standards (restricted).
  - Drop Linux capabilities, run as non-root.

5. **Secrets Management:**
  - Use external secret stores like Vault, AWS Secret Manager


#### Q24: <kbd>How would you implement multi-tenancy in Kubernetes?</kbd>
True multi-tenancy is complex. You can simulate it using:
1. **Namespaces:** Logical isolation, each team/project gets its own namespace.

2. **ResourceQuotas and LimitRanges:** Prevent noisy neighbor issues.

3. **RBAC:** Restrict what users can do in each namespace.

4. **NetworkPolicies:** Enforce network isolation between tenants.

5. **PodSecurityAdmission:** Enforce security policies per tenant.

6. **Custom Admission Controllers:** Add business logic (e.g., image whitelist, label enforcement).

Alternatives:
- Consider Virtual Clusters or tools like vCluster, Loft, or OpenShift Projects for stronger isolation.


---

## Observability

#### Q25: <kbd>How do you implement observability in Kubernetes?</kbd>
- **Logging**: Logs shipped via sidecar or DaemonSets collecting logs from `/var/log/pods/` or `/var/log/containers/` on each node. Fluentd, Fluent Bit, Loki.
- **Metrics**: Cluster and application metrics. Prometheus + kube-state-metrics / node exporter + Grafana.
- **Tracing**: Distributed tracing for microservices. OpenTelemetry, Jaeger, Tempo.
- **Events**: `kubectl get events --sort-by='.lastTimestamp'`


---

## Service Mesh

#### Q26: <kbd>What is a Service Mesh?</kbd>
A Service Mesh adds transparency, observability, and control over microservice communication.

Key features:
- Traffic routing and splitting
- mTLS (encryption in transit)
- Retries, timeouts, circuit breaking
- Metrics and tracing
- Access control and policy

Istio Architecture:
- Sidecar proxy (usually Envoy) injected into each Pod via admission webhook.
- A istiod control plane manages configurations
- Define VirtualService and DestinationRule to control traffic.
- Use istioctl, Prometheus, and Kiali for observability.


---

## Virtualization

#### Q27: <kbd>What is KubeVirt?</kbd>
- Enables running Virtual Machines on Kubernetes.
- Uses CRDs like `VirtualMachine`, `VirtualMachineInstance`.
- Components:
  - `virt-api`: Exposes Kubernetes-style API for VMs
  - `virt-controller`: Manages VM lifecycle
  - `virt-launcher`: Per-VM Pod that wraps QEMU/libvirt
  - `virt-handler`: Runs on each node, manages KVM/qemu processes
  - `libvirt + QEMU`: Underlying VM hypervisor

Networking Options:
- Bridge, masquerade (NAT), passthrough
- SR-IOV for performance

Storage:
- Uses Kubernetes PVCs for VM disks
- Supports live migration with block storage


---

## Troubleshooting

#### Q28: <kbd>How do you debug CrashLoopBackOff?</kbd>
1. Get logs:
```bash
kubectl logs <pod> --previous
```
2. Describe the pod:
```bash
kubectl describe pod <pod>
```
3. Check exit codes:
- 1: App crashed
- 137: OOMKilled


#### Q29: <kbd>What happens when a node fails?</kbd>
- kubelet on each node sends heartbeats to the API Server via the Node Controller.
- If a node fails to report for `node-monitor-grace-period` (default 40s), it's marked `NotReady`.
- After `pod-eviction-timeout` (default 5 minutes), pods are scheduled elsewhere.
- DaemonSet pods are not rescheduled.
- Pods with PodDisruptionBudgets may be protected from eviction.


---

## Miscellaneous

#### Q30: <kbd>What is GitOps?</kbd>
- Git is the source of truth.
- All changes are made via Git commits.
- Version-controlled infrastructure, Easy rollback, Audit-friendly
- Tools: Argo CD, Flux


#### Q31: <kbd>How do you manage multiple clusters?</kbd>
- Challenges: Authentication, Workload placement, Cluster visibility, Networking between clusters.
- Tools: Rancher, Anthos, OpenShift ACM, Amazon EKS Anywhere.
- KubeFed for federated deployments (less common).
- Use `kubectl config use-context` for CLI switching.