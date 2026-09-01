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

## Still Pending

- [ ] Observe status transitions with `kubectl get --watch`.
- [ ] Inspect Pod details and events with `kubectl describe`.
- [ ] State the root cause using the collected evidence.
- [ ] Restore `nginx:1.27` and apply the manifest.
- [ ] Verify that the Pod becomes ready and nginx starts normally.
- [ ] Complete the incident record and learner explanation.

No `ErrImagePull` or `ImagePullBackOff` result has been recorded yet.
