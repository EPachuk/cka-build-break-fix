# Incident 001: Invalid Image Reference

**Status:** Resolved

## Intended Failure

The Pod image was deliberately changed from `nginx:1.27` to `nginx:cka-intentionally-missing` to study how Kubernetes reports and retries an image-pull failure.

## Baseline

Before the change, Pod `web` was `1/1 Running`, had zero restarts, and had been running for `2d15h`.

## Change Applied

The scoped Git diff showed only the intended image-field change. Applying the manifest returned:

```text
pod/web configured
```

## Evidence Collected

- The Pod was healthy before the change.
- The API Server accepted the updated desired state.
- `kubectl get --watch` showed `0/1 ImagePullBackOff` with zero restarts.
- `kubectl describe` showed the container `Waiting`, `Ready: False`, and the Pod still scheduled and initialized.
- A refreshed invalid tag exposed `ErrImagePull` before returning to `ImagePullBackOff`.
- Kubelet events reported backoff while pulling both intentionally invalid image references.
- The original detailed registry response expired before collection; only aggregated `BackOff` and `Failed` events remained.

## Root Cause

The Pod specification requested an intentionally invalid nginx tag. Kubelet could not obtain that reference, reported `ErrImagePull`, and applied `ImagePullBackOff` between retries. The Pod was scheduled and initialized, which excluded scheduling and basic Pod setup as the failing stage.

## Fix

Restored the manifest to `nginx:1.27` and applied the corrected desired state.

## Verification

- Pod returned to `1/1 Running`.
- Pod age remained `3d1h`, confirming that the existing Pod object was updated rather than recreated.
- Restart count became `1` after the new container execution started.
- Current nginx logs showed version 1.27.5 completing configuration, starting worker processes, and producing no errors.

## Lessons Learned

- API acceptance does not prove workload health; state, events, readiness, and application logs must be checked separately.
- Pod phase `Running` does not guarantee that its containers are ready.
- `ErrImagePull` describes a failed attempt; `ImagePullBackOff` describes delayed retries after failures.
- A pull failure can leave restart count at zero because the replacement container never starts.
- Event retention can remove the detailed registry response, so investigation should begin promptly.
- Restoring the image can start a new container within the same Pod, preserving Pod age while increasing restart count.
