# Lab Roadmap

This roadmap organizes the project into work sessions. A day is complete only when the practical work, evidence, documentation, and learner explanation are complete. The order can stretch across multiple calendar days when a topic needs more practice.

## How to Use This Checklist

For every session:

- [ ] Explain the objective and the Kubernetes concepts before running commands.
- [ ] Execute the commands in the indicated environment.
- [ ] Inspect the resulting state instead of assuming success.
- [ ] Record the relevant manifests, observations, and failures in the repository.
- [ ] Explain what changed, why it changed, and how it was verified.
- [ ] Repeat or investigate anything that is not yet understood.

PowerShell is the default shell for this Windows-based lab. Ubuntu in WSL2 is used when Linux-specific work is part of the session. Cluster commands are run by the learner; the guide does not execute practical changes silently.

## Phase 1: Environment and First Workload

### Day 01 - Environment and Cluster

- [x] Review the local architecture: Windows, WSL2, Ubuntu, Docker Desktop, `kind`, and `kubectl`.
- [x] Separate standard Kubernetes concepts from lab-specific implementation choices.
- [x] Explain why `kind` is a deliberate simplification and what it does not reproduce from production.
- [x] Verify the installed tools and Docker Engine.
- [x] Explain what a Kubernetes cluster contains at a high level.
- [x] Create the `kind` cluster from PowerShell.
- [x] Inspect the Node and system Pods.
- [x] Inspect namespaces explicitly and verify the active `kubectl` context.
- [x] Record the cluster creation and observations.

**Evidence:** a reachable cluster, a valid `kubectl` context, and an explanation of the control plane and worker role.

**Day 01 result:** Complete. See [docs/day-01-observations.md](day-01-observations.md) for the commands, evidence, and final mental model.

### Future Infrastructure Stage - Linux VM Cluster

After the Kubernetes operations stage, the project can replace the `kind` implementation with a cluster built on Linux virtual machines. This is a separate stage, not a requirement for the first workload labs.

- [ ] Create or provision Linux VMs with clearly separated control-plane and worker roles.
- [ ] Explain what changes when a Node is a VM instead of a `kind` container.
- [ ] Practice SSH access, Linux services, `containerd`, disks, and node-level logs.
- [ ] Install and configure Kubernetes with a tool such as `kubeadm`.
- [ ] Configure the CNI and verify node-to-node and Pod networking.
- [ ] Compare the VM-based cluster with the local `kind` cluster.
- [ ] Document which Kubernetes concepts stayed the same and which infrastructure concerns changed.

**Scope:** this stage adds infrastructure realism without replacing the Kubernetes concepts already learned with `kind`.

### Day 02 - First Pod

- [x] Explain Pod, container, image, namespace, and declarative manifest.
- [x] Create a Pod from YAML in the repository.
- [x] Use `kubectl get`, `describe`, `logs`, and `exec` to inspect it.
- [x] Compare the desired state in YAML with the observed state.
- [x] Document the Pod lifecycle and useful inspection commands.

**Evidence:** a working Pod manifest and an inspection record.

**Day 02 result:** Complete. The `web` Pod is `Running` on `cka-lab-control-plane`; its logs, container environment, desired state, and observed state were inspected. See [docs/day-02-observations.md](day-02-observations.md) for the evidence and final mental model.

### Day 03 - Break and Diagnose `ImagePullBackOff`

- [x] Explain what happens when a node cannot pull an image.
- [x] Change the Pod to use an intentionally invalid image.
- [x] Observe the status and events over time.
- [x] Diagnose the root cause with `describe` and events.
- [x] Fix the image and verify recovery.
- [x] Write the incident using the Build-Break-Fix format.

**Evidence:** a complete incident with symptom, evidence, root cause, fix, and verification.

**Day 03 result:** Complete. The invalid image produced `ErrImagePull` and `ImagePullBackOff`; kubelet events identified the failed image-pull stage. Restoring `nginx:1.27` returned the Pod to `1/1 Running`, and nginx startup logs verified application recovery. See [docs/day-03-observations.md](day-03-observations.md) and [incidents/001-image-pull-backoff.md](../incidents/001-image-pull-backoff.md).

### Day 04 - Deployments and ReplicaSets

- [ ] Explain why a Deployment manages ReplicaSets and Pods.
- [ ] Create a Deployment with multiple replicas.
- [ ] Scale it and observe the controller reconcile state.
- [ ] Inspect labels, selectors, and ownership references.
- [ ] Break a selector or image configuration in a controlled way and diagnose it.

**Evidence:** a Deployment manifest and a controller observation record.

### Day 05 - Rollouts and Rollbacks

- [ ] Explain revision history and rolling updates.
- [ ] Update an image and observe rollout status.
- [ ] Inspect rollout history and the generated ReplicaSets.
- [ ] Introduce a bad version and detect the failed rollout.
- [ ] Roll back and verify application recovery.

**Evidence:** rollout and rollback commands with before/after verification.

## Phase 2: Configuration, Health, and Scheduling

### Day 06 - ConfigMaps and Secrets

- [ ] Explain configuration versus application image content.
- [ ] Create a ConfigMap and consume it as an environment variable.
- [ ] Create a Secret and consume it safely from a Pod.
- [ ] Inspect references without exposing secret values unnecessarily.
- [ ] Diagnose a missing key or incorrect reference.

**Evidence:** manifests and an explanation of configuration injection.

### Day 07 - Health Probes and Resources

- [ ] Explain startup, liveness, and readiness probes.
- [ ] Add probes and observe their different effects.
- [ ] Configure CPU and memory requests and limits.
- [ ] Create a controlled probe failure and diagnose it.
- [ ] Explain why readiness affects traffic while liveness affects restarts.

**Evidence:** probe results, events, and resource configuration notes.

### Day 08 - Scheduling

- [ ] Explain how the scheduler selects a node.
- [ ] Inspect node labels, taints, and tolerations.
- [ ] Use node selectors and node affinity.
- [ ] Use pod affinity or anti-affinity for placement rules.
- [ ] Create an unschedulable Pod and diagnose the scheduling events.

**Evidence:** scheduling manifests and a diagnosis based on scheduler events.

### Day 09 - Jobs and CronJobs

- [ ] Explain the difference between a long-running workload and a finite task.
- [ ] Create a Job and inspect completion status.
- [ ] Create a CronJob and inspect generated Jobs.
- [ ] Diagnose a failed Job from Pod status and logs.
- [ ] Clean up completed resources deliberately.

**Evidence:** successful and failed batch workload observations.

## Phase 3: Services and Networking

### Day 10 - Services and Discovery

- [ ] Explain how a Service provides a stable virtual endpoint.
- [ ] Create a ClusterIP Service for a Deployment.
- [ ] Inspect selectors, endpoints, and DNS discovery.
- [ ] Test access from another Pod.
- [ ] Break a selector and diagnose missing endpoints.

**Evidence:** Service, EndpointSlice, DNS, and connectivity checks.

### Day 11 - Service Exposure

- [ ] Compare ClusterIP, NodePort, and LoadBalancer behavior.
- [ ] Expose a workload locally and identify the access path.
- [ ] Inspect ports, target ports, and node ports.
- [ ] Diagnose a port mismatch.

**Evidence:** a documented request path from client to Pod.

### Day 12 - Network Policies

- [ ] Explain default allow behavior and policy selection.
- [ ] Apply a deny policy in a test namespace.
- [ ] Add the minimum allow rules needed for application traffic.
- [ ] Test allowed and denied connections.
- [ ] Diagnose a policy problem from labels and connectivity tests.

**Evidence:** policy manifests and reproducible network test results.

## Phase 4: Storage

### Day 13 - Volumes and Persistent Storage

- [ ] Explain ephemeral storage and why Pods are not durable by default.
- [ ] Use an `emptyDir` volume and observe its lifecycle.
- [ ] Inspect the local StorageClass and persistent volume behavior.
- [ ] Create a PersistentVolumeClaim.
- [ ] Mount persistent storage into a workload and verify data behavior.

**Evidence:** storage manifests and a persistence test.

### Day 14 - Storage Troubleshooting

- [ ] Create a PVC that cannot bind and inspect events.
- [ ] Diagnose StorageClass, access mode, and capacity mismatches.
- [ ] Fix the claim and verify binding and mounting.
- [ ] Document the storage troubleshooting sequence.

**Evidence:** a storage incident with root cause and recovery verification.

## Phase 5: Cluster Administration and Security

### Day 15 - Cluster Administration

- [ ] Inspect control-plane components and system namespaces.
- [ ] Explain API Server, scheduler, controller manager, kubelet, and etcd roles.
- [ ] Practice namespaces, resource quotas, and limit ranges.
- [ ] Inspect cluster events and resource usage.
- [ ] Review context switching and safe command targeting.

**Evidence:** a cluster map and a safe administration checklist.

### Day 16 - Access Control and Service Accounts

- [ ] Explain authentication, authorization, and admission at a high level.
- [ ] Create a ServiceAccount.
- [ ] Create a Role and RoleBinding with least privilege.
- [ ] Verify allowed and denied actions with authorization checks.
- [ ] Diagnose an RBAC permission error.

**Evidence:** RBAC manifests and permission verification.

### Day 17 - Security Contexts and Images

- [ ] Explain container user identity and security context settings.
- [ ] Run a workload as a non-root user where supported.
- [ ] Inspect filesystem and privilege behavior.
- [ ] Review image tags, pull policy, and registry assumptions.
- [ ] Diagnose an image or permission failure.

**Evidence:** security configuration and a controlled failure analysis.

## Phase 6: Integration and Exam Practice

### Day 18 - Helm and Repeatable Releases

- [ ] Explain chart, values, release, and rendered manifests.
- [ ] Inspect a chart before installing it.
- [ ] Install a small release with Helm.
- [ ] Inspect the Kubernetes resources created by the release.
- [ ] Upgrade, roll back, and uninstall deliberately.

**Evidence:** release history and a comparison between values and rendered resources.

### Day 19 - Full Build-Break-Fix Scenario

- [ ] Build a small application with Deployment, Service, ConfigMap, Secret, probes, and storage.
- [ ] Define expected behavior and verification commands.
- [ ] Introduce multiple failures one at a time.
- [ ] Diagnose each failure from state, events, logs, and configuration.
- [ ] Restore the application and document the complete incident.

**Evidence:** a complete end-to-end lab and incident report.

### Day 20 - CKA-Style Review

- [ ] Perform timed exercises across workloads, networking, storage, scheduling, and troubleshooting.
- [ ] Work from symptoms and requirements rather than memorized command sequences.
- [ ] Use documentation efficiently and verify every change.
- [ ] Record weak areas and repeat the corresponding sessions.
- [ ] Explain the architecture and troubleshooting method without prompts.

**Evidence:** a review log, identified gaps, and a repeat plan.

## Completion Criteria

The project is complete when the learner can independently:

- [ ] Explain the local environment and the Kubernetes control path.
- [ ] Create, inspect, break, diagnose, fix, and verify workloads.
- [ ] Work with configuration, health, scheduling, networking, and storage.
- [ ] Apply basic cluster administration and least-privilege access control.
- [ ] Read Kubernetes state, events, logs, and manifests to find root causes.
- [ ] Document incidents and explain the reasoning behind each action.
- [ ] Reproduce the labs without relying on an unexplained command sequence.
