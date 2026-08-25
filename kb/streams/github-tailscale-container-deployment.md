# GitHub–Tailscale Container Deployment

**Status: SUPERSEDED**

This container-only stream has been generalized into the canonical procedure:

[GitHub–Tailscale Immutable Artifact Deployment](github-tailscale-artifact-deployment.md)

The new stream preserves the container/Docker Compose profile and adds native binary/systemd and other immutable-artifact deployment profiles under the same invariant:

> Build once → immutable artifact → staging → verify → approve → promote the exact same artifact → production.
