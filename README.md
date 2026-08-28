# Kubernetes Build-Break-Fix Lab

A hands-on Kubernetes administration lab built around the CKA curriculum.

The project follows a repeatable loop:

```text
BUILD -> VERIFY -> BREAK -> OBSERVE -> DIAGNOSE -> FIX -> VERIFY AGAIN
```

The goal is to build practical skill with Kubernetes workloads, networking, storage, scheduling, cluster administration, and troubleshooting under realistic conditions.

## Current Focus

The first stage covers Kubernetes fundamentals and workloads:

- local cluster setup;
- Pods and containers;
- `kubectl` inspection and basic operations;
- declarative YAML;
- Deployments and ReplicaSets;
- ConfigMaps and Secrets;
- health probes;
- rollouts, rollbacks, and incident analysis.

The first lab mission is to create a local cluster, deploy and inspect a Pod, then intentionally trigger and diagnose an `ImagePullBackOff`.

## Repository Structure

```text
cka-build-break-fix/
├── README.md
├── docs/
│   └── learning-log.md
├── labs/
├── incidents/
├── app/
└── backlog/
```

The structure will expand as new CKA domains are covered.

## Learning Method

Every lab should answer:

- What resource is being created or inspected?
- Which namespace and cluster context are involved?
- What result is expected?
- How is the result verified?
- What can fail, and how is the failure diagnosed?

Incidents are documented with the symptom, observations, investigation, root cause, fix, verification, and lessons learned.

## CKA Domains

The project will progressively cover:

- Cluster Architecture, Installation & Configuration
- Workloads & Scheduling
- Services & Networking
- Storage
- Troubleshooting

## Status

Stage 1: local environment setup in progress. The repository and initial documentation are published; create the first local cluster before starting the first lab.

## Environment

The planned local environment uses Docker Desktop with WSL2 and Ubuntu as the Linux foundation, plus `kubectl`, `kind`, and Helm. This combination provides a quick local cluster while keeping the Linux and container fundamentals visible for later CKA labs.
