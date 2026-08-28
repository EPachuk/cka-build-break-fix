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

The primary goal is learning, not merely completing commands. A lab is incomplete until the learner can explain what was done, why each component or command was needed, what state it produced, and how the result was verified. Each new step should be introduced with its purpose and checked through the learner's own explanation before moving on.

## Working Model

The learner executes the commands and owns the resulting environment. The guide explains the purpose and expected evidence before each step, reviews the output afterward, and checks understanding before continuing.

- **PowerShell** is used for Windows, Docker Desktop, WSL, `kind`, and general `kubectl` commands.
- **Ubuntu in WSL2** is used when a Linux shell or Linux-specific investigation is needed.
- **The repository** contains project manifests, lab instructions, observations, and incident records.
- **The chat** is used for explanations, questions, output review, and diagnosis; practical cluster changes are not performed silently on the learner's behalf.

The local architecture is:

```text
Windows -> Docker Desktop and its WSL2/Linux backend -> Docker Engine
		 -> kind node containers -> Kubernetes API Server
		 -> kubectl connects to the API Server through kubeconfig
```

This setup is realistic for practicing the Kubernetes API, resources, contexts, manifests, and troubleshooting. It simplifies production infrastructure because the nodes are Docker containers rather than independent servers.

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
