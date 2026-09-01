# Day 03 Observations: Image Pull Failure

## Objective

Introduce an invalid image reference deliberately, observe the resulting Pod state over time, diagnose the failure from Kubernetes evidence, restore the valid image, and verify recovery.

## Image Pull Model

```text
Pod spec
  -> scheduler assigns a Node
  -> kubelet reads the requested image
  -> kubelet asks containerd to obtain it
  -> containerd contacts the image registry
  -> the image is downloaded
  -> the container is created and started
```

The Kubernetes behavior is standard. The lab-specific implementation is that the Node `cka-lab-control-plane` runs as a Docker container created by `kind`.

`ErrImagePull` reports a failed pull attempt. `ImagePullBackOff` means kubelet is delaying another attempt after repeated failures; it does not mean Kubernetes has stopped retrying permanently.

## Healthy Baseline

Command:

```powershell
kubectl get pod web --namespace default --context kind-cka-lab
```

Observed before introducing the failure:

```text
NAME   READY   STATUS    RESTARTS   AGE
web    1/1     Running   0          2d15h
```

This established that the container was ready, the Pod was running, and no restart problem existed before the controlled change.

## Controlled Change

The image in `labs/01-pods/web-pod.yaml` was changed from:

```yaml
image: nginx:1.27
```

to:

```yaml
image: nginx:cka-intentionally-missing
```

A path-scoped `git diff` confirmed that this was the only change to the manifest.

The new desired state was submitted with:

```powershell
kubectl apply -f .\labs\01-pods\web-pod.yaml --context kind-cka-lab
```

Observed:

```text
pod/web configured
```

This proves that the API Server accepted the updated Pod specification. It does not prove that the Node found the image or started a container.

## Observe the Failure

```powershell
kubectl get pod web --namespace default --context kind-cka-lab --watch
```

Observed:

```text
NAME   READY   STATUS             RESTARTS   AGE
web    0/1     ImagePullBackOff   0          3d1h
```

The Pod object remained present and scheduled, but its only container was not ready. The zero restart count showed that the replacement container had not started and then crashed; image acquisition failed before a new process could run. The Pod age did not reset because the existing Pod was updated rather than recreated.

## Inspect Container State and Events

```powershell
kubectl describe pod web --namespace default --context kind-cka-lab
```

The Pod phase was `Running`, while the container state was `Waiting` with reason `ImagePullBackOff`. `Ready` and `ContainersReady` were `False`; `PodScheduled` remained `True`. This distinction showed that the Pod was assigned and managed but the application was unavailable.

The last container state was `Terminated` with reason `Completed` and exit code `0`. Its container ID and image digest belonged to the previously healthy nginx execution, not to the invalid image.

The retained kubelet events showed repeated backoff over nine hours:

```text
Back-off pulling image "nginx:cka-intentionally-missing"
Error: ImagePullBackOff
```

Events were also queried directly:

```powershell
kubectl get events --namespace default --context kind-cka-lab --field-selector involvedObject.name=web --sort-by=.lastTimestamp
```

Only the aggregated `BackOff` and `Failed` events remained. The original detailed registry response had expired.

To create a fresh observation, the invalid tag was changed to `nginx:cka-intentionally-missing-v2` and applied. A new `describe` captured `State: Waiting` with `Reason: ErrImagePull`. Watching events then captured:

```powershell
kubectl get events --namespace default --context kind-cka-lab --field-selector involvedObject.name=web --watch
```

```text
Back-off pulling image "nginx:cka-intentionally-missing-v2"
Error: ImagePullBackOff
```

The exact registry message, such as `not found`, was not retained. The diagnosis therefore relies on the controlled invalid reference, the waiting reasons, and kubelet events that identify image pulling as the failed stage.

## Root Cause

The Pod specification requested the intentionally invalid nginx tag `cka-intentionally-missing`, later refreshed as `cka-intentionally-missing-v2`. Kubelet could not obtain the requested image, so it reported `ErrImagePull`, delayed repeated attempts with `ImagePullBackOff`, and never started the replacement container.

Scheduling, Pod initialization, and the Pod network were not the cause: the Pod remained scheduled, initialized, and assigned an IP throughout the incident.

## Fix and Recovery

The manifest was restored to:

```yaml
image: nginx:1.27
```

The corrected desired state was applied:

```powershell
kubectl apply -f .\labs\01-pods\web-pod.yaml --context kind-cka-lab
```

Recovery was observed with `kubectl get --watch`:

```text
NAME   READY   STATUS    RESTARTS      AGE
web    1/1     Running   1 (10h ago)   3d1h
```

The unchanged age confirmed that the same Pod object remained. The restart count increased because kubelet started a new container execution within that Pod after the previous nginx container had been terminated during the image update.

Final application verification used:

```powershell
kubectl logs web --namespace default --context kind-cka-lab
```

The logs showed nginx 1.27.5 completing its entrypoint configuration at `2026/09/01 02:53:26`, starting its worker processes, and reporting no errors.

## Day 03 Result

Day 03 is complete. A healthy baseline was recorded, the image was broken deliberately, `ErrImagePull` and `ImagePullBackOff` were observed, kubelet events isolated the failing stage, the valid image was restored, and both Kubernetes readiness and nginx logs verified recovery.
