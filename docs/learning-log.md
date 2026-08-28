# Learning Log

Short notes from each working session. The focus is on what was built, what broke, and what can now be done without help.

## Day 01

### Built

- Repository foundation.
- GitHub repository: `EPachuk/cka-build-break-fix`.
- Local repository at `CKA Project/cka-build-break-fix`, connected to `origin` and using `main`.
- Initial project documentation and `.gitignore`.
- WSL2 base installation and `VirtualMachinePlatform` feature enabled.
- Docker Desktop installation completed.
- Git installed for version control and repository work.
- `kubectl` 1.37.0, `kind` 0.33.0, and Helm 4.2.4 installed with `winget`.
- Ubuntu installation started for WSL2.

### New concepts

- Kubernetes Build-Break-Fix workflow.
- A local Kubernetes cluster needs a container runtime, Kubernetes CLI, and a cluster tool.
- WSL2 provides the Linux and virtualization foundation for the local environment.

### Environment setup explained

The installation is part of the learning material. Nothing in the setup should be treated as a command to run blindly:

- **WSL2** is Windows Subsystem for Linux version 2. It provides a Linux environment inside Windows, backed by a lightweight virtual machine and a real Linux kernel. It is useful here because Kubernetes and container tooling are built around Linux primitives, while the lab is being run from Windows.
- **VirtualMachinePlatform** is a Windows feature that enables the virtualization layer WSL2 uses. It is infrastructure for WSL2, not Kubernetes and not a Kubernetes cluster. A reboot is required after enabling it so Windows can activate the feature.
- **Docker Desktop** runs the Docker Engine and container runtime on Windows, using its Linux/WSL2 backend. `kind` uses this engine to create Kubernetes nodes as containers. Docker Desktop therefore supplies the runtime; it does not by itself create a Kubernetes cluster.
- **Ubuntu** is a Linux distribution installed inside WSL2. It is stored and managed by WSL, separate from this Git repository. It provides a Linux user space for practicing Linux and container commands; its installation does not create Kubernetes resources.
- **`kubectl`** is the Kubernetes command-line client. It sends requests to a Kubernetes API server using the current kubeconfig context. Installing it gives us a way to control a cluster, but there is no cluster until one is created and configured.
- **`kind`** means Kubernetes IN Docker. It creates a local Kubernetes cluster by running Kubernetes node components in Docker containers. It is the tool for creating the disposable practice cluster; running `kind` itself is not the same as installing Kubernetes system-wide.
- **Helm** is a package manager and release tool for Kubernetes applications. It is not required to create the first Pod, but it will be useful later for installing repeatable application stacks and learning chart-based deployments.
- **Git** is version control software used to track manifests, documentation, and incident history. It is a project-development tool, not a Kubernetes component and not required for a cluster to run.

### What “environment state” means

The environment state is a snapshot of which prerequisites are installed, which services are running, which Kubernetes context is selected, and whether a cluster is reachable. For this lab, the checks answer different questions:

- `wsl --status` and `wsl --list --verbose` verify WSL2 and its distributions.
- `docker version` verifies that the Docker client can reach the Docker Engine.
- `kubectl version --client`, `kind version`, and `helm version` verify the local tools.
- `kind get clusters`, `kubectl config current-context`, and `kubectl cluster-info` verify whether a Kubernetes cluster exists and whether `kubectl` can reach it.

The supporting environment and the `kind` cluster are now ready, but no application Pod has been created yet. Creating the cluster and deploying the first Pod are deliberate lab actions, not silent installation steps.

Creating the cluster also started standard Kubernetes components and supporting services: the API Server, scheduler, controller manager, `etcd`, kubelet, CoreDNS, the CNI network plugin, kube-proxy, and a local storage provisioner. Their Kubernetes roles are general; the lab-specific part is that `kind` hosts the Node and these components inside a Docker container instead of using independent production servers.

### How the practice will work

The learner runs each command in the indicated environment and brings the result back for review. PowerShell is the default shell for this Windows-based setup. Ubuntu in WSL2 is used when the exercise specifically requires a Linux shell or Linux investigation. The repository is used for manifests and project documentation; private machine-state notes remain outside it.

Before each action, explain its purpose and expected result. After each action, inspect the actual state and ask the learner to explain what changed. The first Kubernetes action was creating a `kind` cluster; that action is now complete, and the first application Pod has not been created yet.

The local communication path is:

```text
kubectl -> kubeconfig -> Kubernetes API Server
					   -> kind control-plane container
Docker Desktop/Engine -> runs the kind node containers
```

This resembles real Kubernetes administration because the client communicates with the API Server, but `kind` simplifies the infrastructure by representing nodes as containers on the local machine.

### Standard Kubernetes versus lab-specific choices

This distinction should be explained before any practical command. The Kubernetes concepts we are learning are not specific to `kind`: a cluster has a control plane and Nodes; the API Server is the entry point; kubelets and a container runtime run on Nodes; Kubernetes manages Pods and other resources; namespaces organize namespaced resources; controllers reconcile actual state with desired state; and `kubectl` uses a kubeconfig context to address a cluster.

The local implementation is specific to this lab. Windows, Docker Desktop, WSL2, Ubuntu, and `kind` are conveniences for running Kubernetes locally. `kind` represents a Node with a Docker container. In a professional environment, a Node is more commonly a physical server or a Linux VM, and the Node is not wrapped in an outer Docker container. A container runtime still runs the containers inside Pods, but that is a different role from using a container to simulate the Node itself.

Using `kind` is appropriate for the first stage because it focuses effort on Kubernetes operation: API access, resources, manifests, scheduling, reconciliation, networking, storage, and troubleshooting. It is less realistic for infrastructure work such as provisioning servers, SSH access, system services, disks, certificates, high availability, and production network boundaries. A later VM-based stage can cover those differences without obscuring the first Kubernetes concepts.

### Why Docker Desktop was installed

Docker Desktop is a lab-specific dependency, not a universal Kubernetes requirement. We installed it because the host is Windows and `kind` needs a container provider. Docker Desktop supplies the Docker Engine that runs the outer container representing the local Node. In a future VM-based stage, Docker Desktop would not be needed for that outer layer: the Linux VM would be the Node itself, and a Kubernetes-compatible runtime such as `containerd` would run the containers inside Pods.

### Security awareness without scope creep

This local setup does not use SSH because we administer Kubernetes through its API Server rather than logging into a separate Linux VM. The API Server is exposed on `127.0.0.1` through a local port, so the lab is intended for access from this computer, not as a production network design. The connection uses HTTPS/TLS and credentials stored in the local kubeconfig.

These credentials and kubeconfig files must remain private and outside Git. Docker Engine access is also highly privileged, and a local container-based node is not the same security boundary as an independent production server. We will call out these risks when they appear, while keeping detailed hardening, PKI, firewall, and production security design outside the current lab scope.

### Commands learned

- `winget install --id Git.Git --exact --scope user`
- `winget install --id Kubernetes.kubectl --exact --scope user`
- `winget install --id Kubernetes.kind --exact --scope user`
- `winget install --id Helm.Helm --exact --scope user`
- `winget install --id Docker.DockerDesktop --exact`
- `wsl --install --distribution Ubuntu --no-launch`
- `wsl --status`
- `wsl --list --verbose`

### What broke

- WSL2 initially reported that it was not installed.
- Enabling `VirtualMachinePlatform` completed successfully but requires a Windows reboot.
- The first environment check triggered the WSL installation prompt and required elevation.
- The current PowerShell session does not yet see the newly installed PATH entries for `kubectl`, `kind`, and Helm.

### Root cause

- WSL2 and its virtualization feature were not present before setup.
- Windows must restart before the optional feature changes become effective.
- A new shell is needed to load the PATH changes made by `winget`.

### What I can now do without help

- Identify the first lab objective and verification steps.
- Explain why Docker Desktop, WSL2, `kubectl`, and `kind` are needed for the first local cluster.
- Explain the difference between the standard Kubernetes architecture and the local `kind` implementation.
- Explain the difference between the `kind` context, the cluster, the Kubernetes Node, and the Docker container representing that Node.

### Current checkpoint

- The `cka-lab` cluster exists and is reachable through the `kind-cka-lab` context.
- The cluster has one Node, `cka-lab-control-plane`, in `Ready` state.
- The system Pods and the local storage provisioner are running.
- Docker shows the `cka-lab-control-plane` container with the API Server port published on `127.0.0.1`.
- The explicit namespace listing, active-context check, and formal Day 01 observation record are still pending.

### Questions / rabbit holes

- Complete the two remaining read-only Day 01 checks.
- Create the formal Day 01 observation record.

### Tomorrow

- From PowerShell, verify the namespaces with `kubectl get namespaces --context kind-cka-lab`.
- From PowerShell, verify the active context with `kubectl config current-context`.
- Record the Day 01 observations and mark those checks complete.
- Begin Day 02 by explaining Pod, container, image, namespace, and declarative manifest.
- Create and inspect the first application Pod.
