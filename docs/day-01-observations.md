# Day 01 Observations: Environment and Cluster

## Objective

Prepare and inspect a local Kubernetes cluster while distinguishing standard Kubernetes concepts from the implementation choices used by this lab.

## Architecture Used

```text
Windows
└── Docker Desktop
    └── Linux/WSL2 backend and Docker Engine
        └── Docker container: cka-lab-control-plane
            └── Kubernetes Node: cka-lab-control-plane
                ├── Control-plane components
                ├── kubelet and container runtime
                └── Kubernetes Pods
```

`kind` is the tool that created and configured the local Kubernetes cluster using Docker. It did not create a separate VM. Docker Desktop provides the local Linux backend and Docker Engine; `kind` uses that engine to run a container that represents the Node.

In production, the Node would more commonly be a physical server or Linux VM. The Kubernetes concepts and API remain the same, but this laboratory simplifies the infrastructure by using a containerized Node.

## Commands and Evidence

### Cluster creation

```powershell
kind create cluster --name cka-lab
```

Result: `kind` created the `cka-lab` cluster, initialized the control plane, installed the CNI and a local StorageClass, and configured the `kind-cka-lab` context for `kubectl`.

### API Server connectivity

```powershell
kubectl cluster-info --context kind-cka-lab
```

Result: the Kubernetes control plane responded at a local HTTPS endpoint on `127.0.0.1`. CoreDNS also responded through the cluster API.

### Node state

```powershell
kubectl get nodes --context kind-cka-lab
```

Observed:

```text
cka-lab-control-plane   Ready   control-plane   v1.37.0
```

The cluster currently has one Kubernetes Node. It has the control-plane role and can also run workloads in this single-Node lab.

### System Pods

```powershell
kubectl get pods --all-namespaces --context kind-cka-lab
```

Nine system and supporting Pods were observed, all ready and running. They included the API Server, scheduler, controller manager, `etcd`, CoreDNS, `kindnet`, kube-proxy, and the local path storage provisioner.

### Docker view

```powershell
docker ps
```

Observed container: `cka-lab-control-plane` using `kindest/node:v1.37.0`, with the API Server port published from the container to `127.0.0.1` on Windows.

Docker sees the outer infrastructure container. Kubernetes sees the Node and the Pods managed inside the cluster. The same name is used by `kind` for convenience, but the Docker container and Kubernetes Node are different objects viewed through different APIs.

### Namespaces

```powershell
kubectl get namespaces --context kind-cka-lab
```

Observed namespaces:

```text
default
kube-node-lease
kube-public
kube-system
local-path-storage
```

Namespaces are logical groupings in the Kubernetes API. They do not contain Nodes physically. A Pod belongs to a namespace and is scheduled onto a Node; those are separate properties.

### Active context

```powershell
kubectl config current-context
```

Observed context: `kind-cka-lab`.

This is the local default target for `kubectl` commands that do not specify `--context`.

## Final Mental Model

```text
kind -> asks Docker to host the local cluster implementation
Docker -> runs the container representing the Node
Kubernetes -> registers and manages the Node and cluster objects
Namespaces -> organize Kubernetes resources logically
Pods -> Kubernetes work units scheduled onto Nodes
Containers -> execute the application processes inside Pods
```

## Scope and Security Notes

The API Server is reachable locally through `127.0.0.1` using HTTPS/TLS and kubeconfig credentials. This is suitable for a local lab, but it is not a production network or security design. `kind` does not reproduce independent servers, SSH-based Node administration, high availability, or all production isolation boundaries.

## Day 01 Result

The local environment and cluster are operational. The architecture, command results, and distinction between Docker, `kind`, Kubernetes, Nodes, namespaces, Pods, and containers have been reviewed. The first application Pod has not been created yet; that begins Day 02.
