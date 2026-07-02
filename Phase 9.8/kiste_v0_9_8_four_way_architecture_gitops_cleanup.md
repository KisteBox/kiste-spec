# Kiste v0.9.8 — Four-Way Architecture Correction and GitOps/Progressive Delivery Cleanup

Status: Architecture addendum  
Release: `0.9.8`  
Theme: Keep Kiste architecture as four-way, make Workspace Manager an optional runtime, move KisteUnit development into SDK, and keep Argo CD as an optional tool provider.

---

## 1. Purpose

This addendum corrects the v0.9.8 architecture split.

Kiste should not become a five-way architecture.

The correct architecture remains a strong four-way split:

```text
1. CLI
2. Python SDK
3. Core Engine
4. KisteUnit SDK
```

The Workspace Manager is not a fifth core layer.

It is an optional runtime/hosting component that uses the Python SDK.

---

## 2. Correct Four-Way Split

```text
CLI
Python SDK
Core Engine
KisteUnit SDK
```

Meaning:

```text
CLI = command surface
Python SDK = public programmatic interface
Core Engine = runs Kiste lifecycle
KisteUnit SDK = develops, tests, packages, and publishes KisteUnits
```

Core rule:

```text
Core runs Kiste.
SDK exposes Kiste.
CLI operates Kiste.
KisteUnit SDK builds KisteUnits.
```

---

## 3. Why Not Five-Way?

Do not define the architecture as:

```text
CLI
Python SDK
Core Engine
KisteUnit SDK
Workspace Manager
```

That mixes two different categories:

```text
software architecture layers
+
deployment/operation components
```

The Workspace Manager is an application/runtime built on top of Kiste, not a core software layer.

Correct framing:

```text
4 software layers
many possible workspace runtimes
```

---

## 4. Workspace Manager Placement

Workspace Manager is an optional runtime/hosting component.

It may be:

```text
local
self-hosted
team-hosted
enterprise-hosted
future cloud-hosted
```

It depends on the Python SDK:

```text
Workspace Manager -> Python SDK -> Core Engine
```

It coordinates workspaces, but it must not own core lifecycle logic.

Workspace Manager may handle:

```text
workspace registry
workspace graph coordination
multi-workspace status
plan storage
review records
monitor reports
unit cache coordination
team/enterprise policy surface
```

Workspace Manager must not:

```text
bypass Core Engine
bypass WorkspaceAccessPolicy
bypass GitUpdatePlan
bypass review
bypass approved deploy
contain KisteUnit development logic
contain tool-specific execution logic
```

---

## 5. Self-Improvement Placement

Self-improvement is not a separate architecture layer.

Self-improvement is a KisteUnit built using the KisteUnit SDK.

Correct rule:

```text
Self-improvement is a use case of the KisteUnit SDK.
```

Self-improvement may:

```text
inspect Kiste specs
inspect Kiste code
detect spec/code drift
propose GitUpdatePlans
propose tests
propose documentation updates
propose release plans
```

Self-improvement must not:

```text
be imported by Core Engine
mutate Kiste directly
bypass GitUpdatePlan
bypass review
auto-merge by default
be required for normal Kiste operation
```

---

## 6. KisteUnit SDK Responsibility

KisteUnit SDK owns development workflows.

It should handle:

```text
unit scaffolding
unit validation
unit test harness
unit packaging
unit documentation generation
unit metadata generation
sample workspace runner
mock WorkspaceContext
mock WorkspaceAccessPolicy
tool adapter development helpers
```

It is used by:

```text
Kiste self-improvement units
third-party KisteUnit developers
internal platform teams
vendors building Kiste-compatible units
```

Core rule:

```text
KisteUnit SDK develops the ecosystem.
Core Engine runs Kiste.
```

---

## 7. Core Engine Responsibility

Core Engine should stay small and pure.

It owns:

```text
schema validation
workspace loading
capability resolution
policy engine
graph building
plan generation
review gate
deploy validation
monitor model
minimum operating capability kernel
```

Core Engine must not own:

```text
unit scaffolding
unit packaging
unit publishing
third-party unit development workflow
self-improvement workflow
tool adapter generator
marketplace/index logic
SDK test harness
Argo CD integration logic
GitHub Actions execution logic
boto3 execution logic
```

---

## 8. Dependency Rules

Allowed:

```text
CLI -> Python SDK -> Core Engine

KisteUnit SDK -> Python SDK -> Core Engine

Workspace Manager -> Python SDK -> Core Engine

KisteUnits -> Python SDK and/or KisteUnit SDK
```

Forbidden:

```text
Core Engine -> CLI
Core Engine -> KisteUnit SDK
Core Engine -> Self-Improvement Unit
Core Engine -> Workspace Manager
Core Engine -> Argo CD
Core Engine -> kubectl wrapper
Core Engine -> GitHub Actions runner
Core Engine -> boto3 executor
CLI -> Core internals directly
KisteUnit SDK -> CLI
Workspace Manager -> CLI
```

Final dependency rule:

```text
Core has no upward dependencies.
Everything else uses Core through the Python SDK.
```

---

## 9. Argo CD Cleanup

Kiste must remove Argo CD from core assumptions.

Argo CD should not define Kiste GitOps or progressive delivery.

Bad model:

```text
Kiste GitOps = Argo CD
Kiste progressive delivery = Argo CD
Kiste deploy depends on Argo CD
```

Correct model:

```text
Kiste GitOps = Kiste Git model + capability contracts
Kiste progressive delivery = Kiste capability model
Argo CD = optional tool provider
```

Rule:

```text
Kiste owns the deployment model.
Argo CD is only one possible tool that can provide GitOps/reconciliation capabilities.
```

---

## 10. Native GitOps Capabilities

Kiste should own GitOps as capabilities.

Add capability group:

```text
gitops.*
promotion.*
environment.*
release.*
```

Recommended capabilities:

```text
gitops.manifest_export
gitops.environment_layout
gitops.sync_plan
gitops.reconcile_plan
gitops.sync_status
gitops.drift
gitops.health
gitops.rollback_plan
gitops.pr_promotion
gitops.change_trace

promotion.dev_to_staging
promotion.staging_to_prod
promotion.policy_gate
promotion.approval_gate
promotion.evidence_required

environment.dev
environment.staging
environment.prod
environment.overlay
environment.freeze_window

release.version
release.tag
release.note
release.rollback
```

Meaning:

```text
Kiste can structure GitOps repos, generate manifests, plan environment promotion, detect drift, and require approval without depending on Argo CD.
```

---

## 11. Native Progressive Delivery Capabilities

Kiste should own progressive delivery as capabilities.

Add capability groups:

```text
progressive.*
rollout.*
traffic.*
analysis.*
rollback.*
```

Recommended capabilities:

```text
progressive.delivery
progressive.strategy
progressive.policy
progressive.evidence_gate
progressive.pause
progressive.promote
progressive.abort

rollout.canary
rollout.blue_green
rollout.recreate
rollout.rolling
rollout.status
rollout.rollback

traffic.split
traffic.shift
traffic.mirror
traffic.health_gate

analysis.metric_check
analysis.error_rate
analysis.latency
analysis.slo_check
analysis.manual_gate

rollback.plan
rollback.execute_approved
rollback.verify
```

Important rule:

```text
These are Kiste capabilities, not Argo CD capabilities.
```

Argo CD, Argo Rollouts, Flagger, Kubernetes Deployment, GitHub Actions, or custom internal tooling may later provide some of these capabilities.

Kiste owns the model.

---

## 12. Argo CD as Optional KisteUnit

Argo CD may be integrated later as a KisteUnit.

Example:

```yaml
apiVersion: kiste.dev/v0.9.8
kind: KisteUnit

metadata:
  name: argocd-gitops-provider
  module: github.com/KisteBox/kiste-unit-argocd
  version: v0.1.0

spec:
  entrypoints:
    top_level:
      allowed: false
    dependency:
      allowed: true

  provides:
    capabilities:
      - gitops.sync_status
      - gitops.health
      - gitops.drift
      - gitops.reconcile_plan
      - rollout.status

  requires:
    capabilities:
      - git.update
      - runtime.kubernetes
      - policy.validate

  git:
    mode: observe-and-reconcile

  policy:
    mutation_allowed: approved-only
    secret_values_allowed: false
```

Boundary:

```text
Argo CD may reconcile.
Argo CD may report sync and health.
Argo CD may be used by Kiste.

But Argo CD does not define Kiste's deployment model.
```

---

## 13. GitOps Modes

Kiste should support two GitOps modes.

### GitOps Export

```text
Kiste generates GitOps-ready files and PRs.
Another reconciler may apply them.
```

### GitOps Managed

```text
Kiste also monitors sync, health, and drift through a provider such as Argo CD, Flux, Kubernetes, or another tool.
```

Default v0.9.8 mode:

```text
GitOps Export
```

Reason:

```text
It is safer.
It does not require Argo CD.
It keeps Kiste useful even without a GitOps reconciler.
```

---

## 14. Updated v0.9.8 Architecture Rule

```text
Kiste Core runs Kiste.
Python SDK exposes Kiste.
CLI operates Kiste.
KisteUnit SDK builds KisteUnits.

Workspace Manager hosts/coordinators workspaces but is not a fifth architecture layer.

Argo CD is not core.
Argo CD is an optional future provider.

Kiste owns GitOps and progressive delivery as capability models.
```

---

## 15. Updated Non-Goals

v0.9.8 does not include:

```text
Argo CD as core dependency
Argo CD as default progressive delivery engine
Argo CD as required GitOps engine
self-improvement inside core
unit development inside core
tool scaffolding inside core
central plugin marketplace
plugin mode
Workspace Manager as a fifth architecture layer
```

---

## 16. Acceptance Criteria

This addendum is accepted only if:

```text
1. v0.9.8 documentation keeps the architecture as four-way.
2. Workspace Manager is documented as optional runtime/hosting component.
3. Self-improvement is documented as a KisteUnit SDK use case.
4. Core Engine does not import KisteUnit SDK.
5. Core Engine does not import Workspace Manager.
6. Core Engine does not import Argo CD.
7. Argo CD is modeled only as optional future KisteUnit/provider.
8. GitOps capabilities are defined independently of Argo CD.
9. Progressive delivery capabilities are defined independently of Argo CD.
10. KisteUnit SDK owns unit development workflows.
```

---

## 17. Final Rule

```text
Do not make Workspace Manager a fifth layer.

Kiste v0.9.8 keeps the strong four-way split:

CLI
Python SDK
Core Engine
KisteUnit SDK

Self-improvement becomes a KisteUnit SDK use case.
Self-hosted manager becomes an optional runtime using the Python SDK.
Argo CD becomes an optional future provider.

Kiste owns GitOps and progressive delivery as capability models.
Core runs Kiste.
SDK builds KisteUnits.
```
