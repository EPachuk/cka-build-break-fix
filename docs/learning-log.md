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

### What “environment state” means

The environment state is a snapshot of which prerequisites are installed, which services are running, which Kubernetes context is selected, and whether a cluster is reachable. For this lab, the checks answer different questions:

- `wsl --status` and `wsl --list --verbose` verify WSL2 and its distributions.
- `docker version` verifies that the Docker client can reach the Docker Engine.
- `kubectl version --client`, `kind version`, and `helm version` verify the local tools.
- `kind get clusters`, `kubectl config current-context`, and `kubectl cluster-info` verify whether a Kubernetes cluster exists and whether `kubectl` can reach it.

At the end of the setup phase, the first group is ready but no `kind` cluster has been created yet. Creating the cluster and deploying the first Pod are deliberate lab actions, not silent installation steps.

### How the practice will work

The learner runs each command in the indicated environment and brings the result back for review. PowerShell is the default shell for this Windows-based setup. Ubuntu in WSL2 is used when the exercise specifically requires a Linux shell or Linux investigation. The repository is used for manifests and project documentation; private machine-state notes remain outside it.

Before each action, explain its purpose and expected result. After each action, inspect the actual state and ask the learner to explain what changed. The first Kubernetes action will be creating a `kind` cluster. Until that command is intentionally run, only the supporting environment exists.

The local communication path is:

```text
kubectl -> kubeconfig -> Kubernetes API Server
					   -> kind control-plane container
Docker Desktop/Engine -> runs the kind node containers
```

This resembles real Kubernetes administration because the client communicates with the API Server, but `kind` simplifies the infrastructure by representing nodes as containers on the local machine.

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

### Questions / rabbit holes

- Confirm whether Docker Desktop is running after the reboot.
- Confirm that Ubuntu finished installing and complete its first-user setup if prompted.

### Tomorrow

- Reboot Windows.
- Open a new PowerShell and verify `wsl --status`, `wsl --list --verbose`, Docker, `kubectl`, `kind`, and Helm.
- Complete Ubuntu initialization if required.
- Create a local cluster with `kind`.
- Deploy and inspect the first Pod.
- Trigger and diagnose `ImagePullBackOff`.
