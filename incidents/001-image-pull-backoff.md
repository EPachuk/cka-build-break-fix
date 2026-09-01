# Incident 001: Invalid Image Reference

**Status:** In progress

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

## Evidence Still Required

- Pod status transitions after the change.
- Pull failure messages from Pod events.
- Evidence-based root cause.
- Corrective change and API acceptance.
- Successful recovery and final application verification.

## Root Cause

Pending diagnosis. The invalid tag is the intended cause, but the incident will not treat that as proven until Kubernetes events report the pull failure.

## Fix

Pending.

## Verification

Pending.

## Lessons Learned

Pending completion of the incident.
