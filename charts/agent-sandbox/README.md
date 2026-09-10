# agent-sandbox

Packages the [kubernetes-sigs/agent-sandbox](https://github.com/kubernetes-sigs/agent-sandbox)
CRDs and controller. Upstream publishes release manifests but no Helm chart,
so this chart is authored here rather than vendored.

`appVersion` tracks the upstream release (currently `v0.5.3`, from the
`sandbox-with-extensions.yaml` asset). The chart `version` is independent and
is bumped for chart changes.

## What it installs

Four CRDs -- `sandboxes.agents.x-k8s.io`, plus
`sandboxtemplates`, `sandboxwarmpools` and `sandboxclaims` under
`extensions.agents.x-k8s.io` -- and the controller (Namespace, ServiceAccount,
Role/ClusterRole + bindings, two Services, Deployment) in
`agent-sandbox-system`.

## Deltas from the upstream manifest

Two intentional differences, both required by SimbianOS SandboxManager:

1. **`app.kubernetes.io/version` label on the Deployment.** SandboxManager's
   readiness gate (`apps/sandbox_manager/agent_sandbox_client.py`,
   `validate_agent_sandbox_controller_ready`) reads this label and compares it
   against `SANDBOX_AGENT_SANDBOX_CONTROLLER_VERSION`, failing closed on any
   mismatch including absent. Upstream sets no version label, which would leave
   the gate permanently unsatisfied.

2. **Resource requests/limits.** Upstream ships none. The defaults here are
   conservative starting values for a controller-runtime manager.

Everything else is upstream's manifest, unmodified.

## Consumed by

SandboxManager compiles `Sandbox` CRs directly for `session-dedicated`
allocation, and `SandboxTemplate` + `SandboxWarmPool` for the pooled modes
(`shared`, `tenant-dedicated`) -- so `extensions: true` is required, not
optional, for pooled allocation to work.

`SANDBOX_AGENT_SANDBOX_CONTROLLER_NAMESPACE`, `..._DEPLOYMENT` and
`..._VERSION` on the SandboxManager side must match `namespace`, the
Deployment name (`agent-sandbox-controller`) and `image.tag` here.

## Note on CRD upgrades

CRDs live in `templates/crds/` rather than the chart-root `crds/` directory, so
`helm upgrade` updates them. The tradeoff is that `helm uninstall` will delete
them, and with them every `Sandbox` object in the cluster. Uninstall
deliberately, not casually.
