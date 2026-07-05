# Kiste v0.9.11 — Boilerplate for Standard KisteUnits, Strong Go Hooks, and Future Four-Repo Split

Status: Architecture preparation  
Release: `0.9.11`  
Theme: Prepare v0.9.12 standard bundled KisteUnits and v0.9.13 four-repo split while keeping the repositories able to live together during development.

---

## 1. Purpose

v0.9.11 is not only about preference and fit.

It must also prepare the project structure for the next two releases:

```text
v0.9.12:
  build a few standard KisteUnits bundled with Kiste Core as a standard library

v0.9.13:
  split Kiste into four clean repositories
```

Therefore v0.9.11 should add boilerplate for:

```text
standard KisteUnit layout
strong Go hook foundation
tool integration unit layout
workspace compatibility layout
future four-repo split
monorepo-friendly development
multi-repo-ready release
```

Core rule:

```text
v0.9.11 prepares the shape.
v0.9.12 fills the standard library.
v0.9.13 splits the repositories.
```

---

## 2. Roadmap Split

### v0.9.11

```text
Prepare boilerplate.
Define strong Go hook contracts.
Define standard unit skeleton.
Define repo boundary markers.
Keep monorepo development possible.
Make four-repo split predictable.
```

### v0.9.12

```text
Build a few standard KisteUnits.
Bundle them with Core as the Kiste standard library.
Keep them capability-first.
Keep them replaceable by external units.
```

### v0.9.13

```text
Split Kiste into four repositories.
Preserve the same package boundaries.
Preserve the same import rules.
Preserve the same unit contracts.
Keep the existing core/spec repository as the anchor repo.
```

---

## 3. Correct Four Repositories

The future four repositories are:

```text
1. kiste-cli
2. kiste-py
3. kiste-unit-sdk
4. kiste-core + spec
```

Meaning:

```text
kiste-cli:
  CLI command surface, terminal UX, command routing, and user-facing executable package

kiste-py:
  Python SDK facade, programmatic API, Python package surface, and developer import path

kiste-unit-sdk:
  builders, Tool Weaver, integration-unit scaffolding, KisteUnit test harnesses, package helpers

kiste-core + spec:
  existing anchor repository for Core Engine and specs
  lifecycle engine, capability graph, hook runtime, policy engine, plan/review/monitor models
  schemas, release specs, capability contracts, and standard bundled KisteUnit definitions
```

Important correction:

```text
There is no separate kiste-standard-units repository in the four-way split.

Standard KisteUnits are bundled with kiste-core + spec as the Kiste standard library.

They may later be extracted if needed, but they are not one of the first four repositories.
```

Optional later repositories:

```text
kiste-go-hooks
kiste-manager
kiste-cloud
kiste-examples
kiste-standard-units, only if the standard library becomes too large later
```

But v0.9.13 should first split the stable four.

---

## 4. Repositories May Live Together First

Before the split, these four repos may live together in one development workspace.

Recommended temporary layout:

```text
kiste-workspace/
  repos/
    kiste-cli/
    kiste-py/
    kiste-unit-sdk/
    kiste-core-spec/
```

Or in a monorepo-compatible layout:

```text
repo-root/
  cli/
  py/
  unit-sdk/
  core/
  spec/
  standard-units/
```

Rule:

```text
The code may live together during development, but boundaries must already behave like four separate repositories.

The existing core/spec repo remains the anchor until the v0.9.13 split is complete.
```

---

## 5. Boundary Markers

v0.9.11 should add boundary markers so the v0.9.13 split is mechanical.

Each future repo area should contain:

```text
README.md
kiste.repo.yaml
pyproject.toml, go.mod, or equivalent package descriptor
OWNERS or CODEOWNERS section
public API boundary
import boundary notes
release boundary notes
```

Example:

```yaml
apiVersion: kiste.dev/v0.9.11
kind: KisteRepoBoundary

metadata:
  name: kiste-core-spec

spec:
  future_repo: github.com/KisteBox/kiste-spec
  current_path: ./
  split_role: core-plus-spec-anchor

  owns:
    - lifecycle
    - capability_graph
    - hook_runtime
    - policy_engine
    - plan_model
    - review_model
    - monitor_model
    - schemas
    - release_specs
    - standard_bundled_units

  must_not_import:
    - kiste-cli
    - kiste-py
    - kiste-unit-sdk
```

---

## 6. Import Rules for Future Split

Allowed:

```text
kiste-cli -> kiste-py
kiste-cli -> kiste-core + spec only through stable CLI-facing APIs if needed
kiste-py -> kiste-core + spec
kiste-unit-sdk -> kiste-core + spec
standard bundled units -> kiste-core + spec contracts
```

Forbidden:

```text
kiste-core + spec -> kiste-cli
kiste-core + spec -> kiste-py
kiste-core + spec -> kiste-unit-sdk
kiste-cli -> kiste-unit-sdk internals
kiste-py -> kiste-unit-sdk internals
standard bundled units -> kiste-cli internals
standard bundled units -> kiste-py internals
```

Rule:

```text
Core has no upward dependencies.
CLI is thin.
Python SDK is the public programmatic facade.
Unit SDK is for building units, not for running Core.
Standard bundled units depend on Core contracts, not Core internals.
```

---

## 7. Strong Go Hook Foundation

v0.9.11 must prepare a strong Go hook foundation because many target tools are Go-native.

Target tools include:

```text
Kubernetes
Docker / Moby / containerd
OCI tools
Helm
Kustomize
Argo CD
Flux
Prometheus
OpenTelemetry Collector
Envoy / Gateway API
cert-manager
External Secrets
Kyverno
OPA / Gatekeeper
Kubeflow
```

Core rule:

```text
Kiste should understand Go-native tools through stable hook contracts, not by rewriting them.
```

---

## 8. Go Hook Boilerplate

Suggested Go hook package layout:

```text
go-hooks/
  kistehook/
    hook.go
    capability.go
    lifecycle.go
    result.go
    policy.go
    manifest.go
    adapter.go

  adapters/
    kubernetes/
    docker/
    oci/
    compose/
    helm/
    kustomize/
    kubeflow/
```

Minimum Go interface shape:

```go
package kistehook

type LifecycleStage string

const (
    ReadStage    LifecycleStage = "read"
    InspectStage LifecycleStage = "inspect"
    PlanStage    LifecycleStage = "plan"
    ReviewStage  LifecycleStage = "review"
    DeployStage  LifecycleStage = "deploy"
    MonitorStage LifecycleStage = "monitor"
)

type HookSpec struct {
    Name     string
    Tool     ToolSpec
    Provides []string
    Requires []string
    Stages   []LifecycleStage
    Mutation MutationPolicy
}

type HookContext struct {
    WorkspacePath string
    Stage         LifecycleStage
    Policy        map[string]any
    ApprovedPlan  map[string]any
}

type HookResult struct {
    Facts          map[string]any
    Findings       []map[string]any
    Plans          []map[string]any
    Reports        []map[string]any
    Evidence       []map[string]any
    MonitorSignals []map[string]any
}

type Hook interface {
    Spec() HookSpec
    Read(ctx HookContext) (HookResult, error)
    Inspect(ctx HookContext) (HookResult, error)
    Plan(ctx HookContext) (HookResult, error)
    Review(ctx HookContext) (HookResult, error)
    Deploy(ctx HookContext) (HookResult, error)
    Monitor(ctx HookContext) (HookResult, error)
}
```

---

## 9. Why Go Hooks Matter

Go hooks matter because the best integrations will often use existing Go APIs and object models.

Examples:

```text
Kubernetes clients are strongest in Go.
Many CNCF tools are written in Go.
Controller-runtime patterns are Go-native.
OCI/container ecosystem is Go-heavy.
Kubeflow and Kubernetes objects fit naturally with Go structs and CRDs.
```

But the Kiste control plane remains capability-first.

```text
Go hook is implementation infrastructure.
Capability remains the control-plane contract.
```

---

## 10. Standard KisteUnits for v0.9.12

v0.9.12 should build a few standard KisteUnits bundled with Core as a standard library.

Recommended first standard units:

```text
standard-unit-git
standard-unit-docker
standard-unit-oci
standard-unit-compose
standard-unit-kubernetes-basic
standard-unit-secret-ref
standard-unit-policy-basic
standard-unit-report-basic
```

Optional but useful:

```text
standard-unit-github-basic
standard-unit-kustomize-basic
standard-unit-helm-render-basic
standard-unit-prometheus-readonly
standard-unit-iam-readonly
```

Rule:

```text
Standard KisteUnits are bundled, trusted defaults inside kiste-core + spec.
They must remain replaceable by external KisteUnits.
```

---

## 11. Standard Library Model

Standard KisteUnits are like a language standard library.

They provide common behavior without forcing users to install everything.

Characteristics:

```text
bundled with Kiste Core
versioned with Kiste
safe defaults
capability-first
policy-aware
replaceable
small surface area
no secret values
no unsafe mutation by default
```

They should not become a giant platform.

---

## 12. Standard Unit Shape

Each standard unit should have:

```text
kiste.unit.yaml
README.md
capabilities.yaml
policy.yaml
examples/
tests/
fixtures/
```

Example:

```yaml
apiVersion: kiste.dev/v0.9.12
kind: KisteUnit

metadata:
  name: standard-unit-kubernetes-basic
  module: std:kubernetes-basic
  version: v0.9.12

spec:
  category: standard-library-unit

  provides:
    capabilities:
      - runtime.kubernetes_lize
      - runtime.manifest_generate
      - runtime.manifest_validation
      - runtime.secret_ref

  policy:
    bundled: true
    bundled_in: kiste-core-spec
    replaceable: true
    mutation_default: false
    approved_plan_required: true
    secret_values_allowed: false
```

---

## 13. Standard Unit Resolution

Standard units should resolve before external units only when they are safest or preferred by workspace policy.

Resolution order should be policy-driven:

```text
1. Core built-in capability
2. Standard bundled KisteUnit from kiste-core + spec
3. Workspace-pinned KisteUnit
4. External KisteUnit
5. Tool-specific integration unit
```

But preference may override this.

Example:

```text
For generic Kubernetes manifests, standard-unit-kubernetes-basic is enough.
For ML pipelines, kiste-unit-kubeflow may be preferred over standard-unit-kubernetes-basic.
```

---

## 14. v0.9.11 Boilerplate Outputs

v0.9.11 should define these future outputs:

```text
.kiste/repo-boundaries/four-repo-boundary-report.json
.kiste/repo-boundaries/import-boundary-report.json
.kiste/repo-boundaries/split-readiness-report.json

.kiste/hooks/go-hook-contract-report.json
.kiste/hooks/go-adapter-boilerplate-report.json

.kiste/standard-units/standard-unit-catalog.json
.kiste/standard-units/standard-unit-readiness-report.json
.kiste/standard-units/bundled-unit-policy-report.json
```

---

## 15. v0.9.11 Acceptance Criteria

v0.9.11 is accepted only if:

```text
1. The correct four future repositories are named.
2. The four repositories are kiste-cli, kiste-py, kiste-unit-sdk, and kiste-core + spec.
3. kiste-core + spec is recognized as the existing anchor repo.
4. There is no separate kiste-standard-units repo in the initial four-way split.
5. Repo boundary markers are defined.
6. Import rules for the future split are defined.
7. Monorepo development remains possible.
8. Multi-repo split is mechanically prepared.
9. Strong Go hook interface is defined.
10. Go-native tool adapters are named.
11. Standard KisteUnit catalog is prepared.
12. v0.9.12 standard-unit scope is defined.
13. v0.9.13 four-repo split scope is defined.
14. Standard units are replaceable by external units.
15. Core does not import CLI, Python SDK, or Unit SDK internals.
```

---

## 16. Final Rule

```text
v0.9.11 prepares the boilerplate.

v0.9.12 builds the standard KisteUnit library bundled with kiste-core + spec.

v0.9.13 splits the repositories four ways:

  1. kiste-cli
  2. kiste-py
  3. kiste-unit-sdk
  4. kiste-core + spec

The four repos may live together during development, but the boundaries must already behave like separate repositories.

Strong Go hooks are prepared in v0.9.11 because Kubernetes, Docker, OCI, Kubeflow, and many CNCF tools are Go-native.
```
