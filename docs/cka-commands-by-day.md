# CKA Commands by Day

This reference collects the Kubernetes commands actually used in the lab that are relevant to CKA-style tasks. It grows as each day is completed.

The CKA is a performance-based command-line exam. This list maps our practice to the published CKA competencies; it does not claim that a specific command or question will appear on an exam.

Excluded deliberately:

- environment installation and verification commands such as `winget`, `wsl`, and tool version checks;
- local lab infrastructure commands from Docker and `kind`;
- repository workflow commands such as `git diff`;
- commands discussed but not yet executed in the lab.

## Day 01 - Cluster Inspection and Targeting

### Inspect cluster endpoints

```powershell
kubectl cluster-info --context kind-cka-lab
```

Used to verify that the Kubernetes API Server is reachable and to display cluster service endpoints. In an exam task, this helps distinguish a connectivity or context problem from a workload problem.

### Inspect Nodes

```powershell
kubectl get nodes --context kind-cka-lab
```

Used to list the cluster Nodes and inspect readiness, roles, age, and Kubernetes version. Node state is a basic starting point for cluster and scheduling troubleshooting.

### Inspect Pods across all namespaces

```powershell
kubectl get pods --all-namespaces --context kind-cka-lab
```

Used to inspect workloads and system components throughout the cluster rather than only in the current namespace. `--all-namespaces` can be abbreviated as `-A`.

### Inspect namespaces

```powershell
kubectl get namespaces --context kind-cka-lab
```

Used to list the logical namespaces available in the cluster. Many exam tasks identify a target namespace explicitly, and resources with the same name can exist in different namespaces.

### Verify the active context

```powershell
kubectl config current-context
```

Used to show which kubeconfig context commands will target when `--context` is omitted. Checking or switching to the required context before making changes is essential in a multi-cluster exam environment.

## Day 02 - Create and Inspect a Pod

### Apply a declarative manifest

```powershell
kubectl apply -f .\labs\01-pods\web-pod.yaml --context kind-cka-lab
```

Used to send the desired state from a YAML file to the API Server. `apply` creates the resource when absent and updates supported fields when it already exists.

### Inspect the Pod summary

```powershell
kubectl get pod web --namespace default --context kind-cka-lab
```

Used for a quick view of container readiness, Pod status, restart count, and age. This is usually the first workload check before deeper diagnosis.

### Inspect placement and network details

```powershell
kubectl get pod web --namespace default --context kind-cka-lab --output wide
```

Used to add details such as the Pod IP and assigned Node. `--output wide` can be abbreviated as `-o wide`.

### Inspect details and events

```powershell
kubectl describe pod web --namespace default --context kind-cka-lab
```

Used to inspect configuration, conditions, container state, placement, and recent events. The events section is especially valuable for diagnosing scheduling, image-pull, mount, and startup failures.

### Read container logs

```powershell
kubectl logs web --namespace default --context kind-cka-lab
```

Used to read the standard output and standard error produced by the Pod's container. Logs help determine whether the application started and whether a failure is occurring inside the process.

### Execute a command inside a container

```powershell
kubectl exec web --namespace default --context kind-cka-lab -- hostname
```

Used to start a short-lived command inside the running container. The `--` separates arguments for `kubectl` from the command executed in the container. This is useful for checking files, environment variables, DNS, identity, and connectivity during troubleshooting.

### Inspect the complete stored object

```powershell
kubectl get pod web --namespace default --context kind-cka-lab --output yaml
```

Used to retrieve the complete Pod object from the API Server. It exposes metadata, desired state in `spec`, defaults added by Kubernetes, and observed state in `status`. `--output yaml` can be abbreviated as `-o yaml`.

## Day 03 - Controlled Image Failure

### Capture the healthy baseline

```powershell
kubectl get pod web --namespace default --context kind-cka-lab
```

Used again before introducing the failure to prove that the Pod was healthy. A baseline prevents an existing problem from being mistaken for the result of the controlled change.

### Apply the invalid image reference

```powershell
kubectl apply -f .\labs\01-pods\web-pod.yaml --context kind-cka-lab
```

Used after changing the manifest to `nginx:cka-intentionally-missing`. The `pod/web configured` response proved only that the API Server accepted the new desired state; it did not prove that the Node could pull or run the image.

### Watch Pod status changes

```powershell
kubectl get pod web --namespace default --context kind-cka-lab --watch
```

Used to keep a read-only request open and print status changes until `Ctrl+C` is pressed. It captured `ImagePullBackOff` during the failure and `1/1 Running` after recovery.

### Diagnose the Pod and inspect events

```powershell
kubectl describe pod web --namespace default --context kind-cka-lab
```

Used to compare Pod phase with container state and readiness, then inspect kubelet events. It exposed `Waiting`, `ErrImagePull` or `ImagePullBackOff`, and `Ready: False` while the Pod remained scheduled.

### List Pod events chronologically

```powershell
kubectl get events --namespace default --context kind-cka-lab --field-selector involvedObject.name=web --sort-by=.lastTimestamp
```

Used to filter events to object `web` and sort them by their last occurrence. This isolates workload evidence from unrelated namespace events.

### Watch Pod events

```powershell
kubectl get events --namespace default --context kind-cka-lab --field-selector involvedObject.name=web --watch
```

Used to capture new kubelet events as they were published. It confirmed backoff while pulling the refreshed invalid image reference.

### Apply the repaired image reference

```powershell
kubectl apply -f .\labs\01-pods\web-pod.yaml --context kind-cka-lab
```

Used after restoring `nginx:1.27`. The API response was followed by a status watch and log inspection because `configured` alone does not prove recovery.

### Verify the recovered application logs

```powershell
kubectl logs web --namespace default --context kind-cka-lab
```

Used to confirm that the recovered nginx process completed configuration and started its workers without errors.

## Common Targeting and Output Flags

| Flag | Purpose |
| --- | --- |
| `--context <name>` | Select the kubeconfig context and therefore the target cluster and credentials. |
| `--namespace <name>` or `-n <name>` | Select the namespace containing a namespaced resource. |
| `--all-namespaces` or `-A` | Query namespaced resources across the whole cluster. |
| `--output wide` or `-o wide` | Show additional summary columns such as Pod IP and Node. |
| `--output yaml` or `-o yaml` | Return the complete API object as YAML. |
| `-f <path>` | Read a resource definition from a file. |
| `--field-selector <expression>` | Filter API objects by supported fields, such as the name of an event's involved object. |
| `--sort-by=<field>` | Sort returned objects by a JSON or field path. |
| `--watch` | Keep the request open and print changes until interrupted. |
| `--` | End `kubectl` arguments and begin the command passed to a container. |

## Exam Practice Rule

Before executing a command, identify the required context, namespace, resource type, resource name, and expected evidence. After executing it, verify the resulting state instead of treating a successful command response as proof that the workload is healthy.

Official references:

- [Certified Kubernetes Administrator](https://www.cncf.io/training/certification/cka/)
- [kubectl command reference](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)
