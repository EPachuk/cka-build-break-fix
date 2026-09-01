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
│   ├── cka-commands-by-day.md
│   ├── day-01-observations.md
│   ├── day-02-observations.md
│   ├── day-03-observations.md
│   ├── lab-roadmap.md
│   └── learning-log.md
├── incidents/
│   └── 001-image-pull-backoff.md
└── labs/
    └── 01-pods/
        └── web-pod.yaml
```

The structure will expand as new CKA domains are covered.

The day-by-day checklist is in [docs/lab-roadmap.md](docs/lab-roadmap.md). It tracks practical work, evidence, and understanding from the first cluster through the final review.

The exam-focused command reference is in [docs/cka-commands-by-day.md](docs/cka-commands-by-day.md). It records only Kubernetes commands actually practiced in the lab and excludes local environment setup.

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

## What Is Standard and What Is Simplified

This distinction must be explicit before each lab step. The following are standard Kubernetes concepts that also apply in professional environments: clusters, control planes, worker nodes, the API Server, kubelets, container runtimes, Pods, namespaces, Services, controllers, desired state, `kubectl`, kubeconfig contexts, manifests, events, logs, and reconciliation.

The following are choices specific to this local lab: Windows as the host, Docker Desktop and WSL2 as the local foundation, and `kind` creating Kubernetes nodes as Docker containers. In production, a Node is more commonly a physical server or a Linux virtual machine, and several independent nodes are used for capacity and availability. A Node does not inherently require Docker as an outer layer.

`kind` is therefore a deliberate simplification, not the definition of Kubernetes. It lets us practice the Kubernetes API and operations without first building a multi-VM infrastructure. It does not fully reproduce server provisioning, SSH administration, system services, disks, high availability, or production network boundaries. Those differences must be named when they matter; they must not be presented as if they were universal Kubernetes behavior.

The project will first use `kind` to learn Kubernetes operations, then can add a VM-based stage to study Node and cluster infrastructure more realistically.

### Why Docker Desktop Is Installed

Docker Desktop is not a universal Kubernetes requirement. It was installed because this lab runs on Windows and uses `kind`; `kind` needs a container provider, and Docker Desktop provides the Docker Engine that runs the container representing the local Node. If we later use Linux VMs as Nodes, Docker Desktop will no longer be needed for that outer layer. The VM will be the Node itself and will use a Kubernetes-compatible container runtime, such as `containerd`, to run Pod containers.

## CKA Domains

The project will progressively cover:

- Cluster Architecture, Installation & Configuration
- Workloads & Scheduling
- Services & Networking
- Storage
- Troubleshooting

## Status

Stage 1: Days 01 through 03 are complete. The `web` Pod was inspected while healthy, changed to request an invalid image, observed in `ErrImagePull` and `ImagePullBackOff`, diagnosed from container state and kubelet events, restored to `nginx:1.27`, and verified as `1/1 Running` with healthy nginx startup logs.

## Environment

The planned local environment uses Docker Desktop with WSL2 and Ubuntu as the Linux foundation, plus `kubectl`, `kind`, and Helm. This combination provides a quick local cluster while keeping the Linux and container fundamentals visible for later CKA labs.
