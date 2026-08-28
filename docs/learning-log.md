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
