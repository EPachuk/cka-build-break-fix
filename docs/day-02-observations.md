# Day 02 Observations: First Pod

## Objective

Create and inspect the first application Pod while connecting the desired state in YAML with the objects and runtime state observed in Kubernetes.

## Manifest

The Pod manifest is [labs/01-pods/web-pod.yaml](../labs/01-pods/web-pod.yaml).

It declares:

- API version `v1` and resource kind `Pod`;
- name `web` in namespace `default`;
- label `app=web`;
- one container named `nginx`;
- image `nginx:1.27`;
- TCP port 80 inside the container.

The manifest was first checked with a client-side dry run. The dry run reported `pod/web created (dry run)` and did not change the cluster.

## Commands and Evidence

### Apply the manifest

```powershell
kubectl apply -f .\labs\01-pods\web-pod.yaml --context kind-cka-lab
```

Result:

```text
pod/web created
```

The API Server accepted and recorded the desired Pod state.

### Check Pod status

```powershell
kubectl get pod web --namespace default --context kind-cka-lab
```

Observed:

```text
web    1/1    Running    0    103s
```

The Pod has one container, that container is ready and running, and it has not restarted.

### Check placement and networking

```powershell
kubectl get pod web --namespace default --context kind-cka-lab --output wide
```

Observed:

```text
web    1/1    Running    0    3m4s    10.244.0.5    cka-lab-control-plane
```

The Pod was assigned to `cka-lab-control-plane` and received the internal Pod IP `10.244.0.5` from the cluster network. This IP is not a public address and no Service or host port was created.

### Describe the Pod

```powershell
kubectl describe pod web --namespace default --context kind-cka-lab
```

Important observations:

- Node: `cka-lab-control-plane` with Node IP `172.18.0.2`;
- Pod IP: `10.244.0.5`;
- container runtime ID begins with `containerd://`;
- image: `nginx:1.27`;
- container port: `80/TCP`;
- host port: `0/TCP`, so the port is not published to Windows;
- all current conditions are positive, including `PodScheduled`, `ContainersReady`, and `Ready`;
- the Pod uses the default ServiceAccount;
- Kubernetes injected a projected, read-only ServiceAccount volume;
- QoS class is `BestEffort` because no CPU or memory requests/limits were defined.

The events showed the lifecycle:

```text
Scheduled -> Pulling -> Pulled -> Created -> Started
```

This means the scheduler selected the Node, the kubelet requested the image, the runtime pulled the image, created the container, and started nginx.

### Read the container logs

```powershell
kubectl logs web --namespace default --context kind-cka-lab
```

The logs showed the official nginx entrypoint applying its startup configuration and then starting nginx 1.27.5. Nginx reported the Linux WSL2 kernel, started its worker processes, and produced no error messages. No HTTP access entries were present because no request had been sent to the Pod.

### Execute a command in the container

```powershell
kubectl exec web --namespace default --context kind-cka-lab -- hostname
```

Observed:

```text
web
```

The command ran as a short-lived process inside the existing `nginx` container and returned the Pod hostname. It did not run in PowerShell, replace nginx, restart the container, or create another Pod. The `--` separator distinguishes the `kubectl` arguments from the command sent to the container.

### Compare desired and observed state

```powershell
kubectl get pod web --namespace default --context kind-cka-lab --output yaml
```

The object returned by the API Server was compared with the repository manifest:

- the repository manifest expresses the requested Pod configuration;
- `spec` contains the desired state, including defaults added by Kubernetes;
- `metadata` identifies and tracks the stored object;
- `status` reports the state observed by Kubernetes;
- the scheduler added `nodeName: cka-lab-control-plane` after selecting the Node;
- Kubernetes added defaults such as `restartPolicy: Always`, `dnsPolicy: ClusterFirst`, and `imagePullPolicy: IfNotPresent`;
- Kubernetes projected the default ServiceAccount credentials into a read-only volume;
- `phase: Running`, `state.running`, `started: true`, `ready: true`, and `restartCount: 0` confirmed that the container was operating normally;
- the requested tag remained `nginx:1.27`, while `imageID` identified the exact downloaded image by digest.

The learner explained the central distinction: the file in the repository states what should be created, while the API Server response shows the stored object, its completed desired state, and its observed runtime status.

## Mental Model

```text
Namespace default
└── Pod web
    ├── Pod IP: 10.244.0.5
    └── Container nginx
        ├── Image: nginx:1.27
        └── Port: 80/TCP inside the container

Node cka-lab-control-plane
└── hosts the Pod
```

The namespace organizes the Pod logically. The Node provides the execution capacity. The Pod is the Kubernetes work unit. The container runs the nginx process from the image.

## Day 02 Result

Day 02 is complete. The Pod was created declaratively and inspected with `get`, `describe`, `logs`, and `exec`. Its desired configuration was compared with the object and runtime status reported by Kubernetes. The intentional `ImagePullBackOff` failure belongs to Day 03 and has not started.
