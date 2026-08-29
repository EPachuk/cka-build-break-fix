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

## Still Pending

- [ ] Read the nginx logs.
- [ ] Execute a command inside the running container with `kubectl exec`.
- [ ] Compare the YAML desired state with the observed state.
- [ ] Document the Pod lifecycle and inspection commands completely.

The intentional `ImagePullBackOff` failure belongs to Day 03 and has not started.
