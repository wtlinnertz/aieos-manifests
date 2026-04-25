# aieos-manifests

Flux-watched manifests repository for AIEOS-deployed services.

## Layout

```
envs/
  dev/      <-- Flux Kustomization for dev reconciles ./envs/dev
  staging/  <-- staging reconciles ./envs/staging
  prod/     <-- prod reconciles ./envs/prod (manual gate from staging)
```

Each environment receives commits from upstream services' `publish.manifest`
adapters and `deploy.environment` (Flux handoff) adapters. The pilot
`aieos-artifact-store` is the v1 reference consumer.

## How services land here

A service's CI runs the `publish.manifest` capability via
`adapter-kustomize-build` after its build + sign pipeline succeeds. The
adapter:

1. Renders the service's `kustomize/overlays/<env>` overlay with the
   freshly-signed image digest.
2. Writes the rendered manifest to `envs/<env>/<service>.yaml` in this
   repo.
3. Commits and pushes.

Flux reconciles the new commit per the `Kustomization` resources in each
service's `flux/kustomization-<env>.yaml`.

## Environment policy

- `envs/dev/` — auto-promote from CI; commits land per successful build.
- `envs/staging/` — auto-promoted from dev (per CD spec promotion edge).
- `envs/prod/` — manual-gate-required; CD spec enforces approval before
  the runner allows the prod-targeted `publish.manifest` step.
