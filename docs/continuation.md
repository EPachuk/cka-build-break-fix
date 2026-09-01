# Continuation Checkpoint

## Current Position

Days 01 and 02 are complete. Day 03 is in progress at the point immediately after applying an intentionally invalid image to Pod `web`.

The manifest and the live Pod specification request:

```yaml
image: nginx:cka-intentionally-missing
```

The last observed command result was:

```text
pod/web configured
```

This only confirms API acceptance. The runtime failure has not yet been observed, diagnosed, or fixed.

## Next Action

Before running the next command, explain that `--watch` keeps a read-only request open and prints state changes until `Ctrl+C` is pressed. Then the learner runs:

```powershell
kubectl get pod web --namespace default --context kind-cka-lab --watch
```

Allow approximately 30 seconds, stop with `Ctrl+C`, and record every displayed status. Do not claim that `ErrImagePull` or `ImagePullBackOff` occurred until the learner provides the output.

## Remaining Day 03 Sequence

1. Interpret the watched status transitions.
2. Explain and run `kubectl describe pod web --namespace default --context kind-cka-lab`.
3. Diagnose the root cause from events rather than from the intentionally chosen filename alone.
4. Change the manifest back to `nginx:1.27`, review the diff, and apply it.
5. Verify `1/1 Running`, inspect recovery events or logs, and complete the incident record.

The learner runs cluster commands. Each command must be explained before execution, its exact output reviewed afterward, and understanding checked before proceeding.
