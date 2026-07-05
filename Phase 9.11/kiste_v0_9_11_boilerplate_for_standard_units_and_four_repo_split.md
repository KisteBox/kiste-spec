# Kiste v0.9.11 — Boilerplate for Standard KisteUnits, Strong Go Hooks, and Future Five-Repo Split

Status: Architecture preparation  
Release: `0.9.11`  
Theme: Prepare v0.9.12 standard bundled KisteUnits and v0.9.13 five-repo split while keeping the repositories able to live together during development.

---

## 1. Purpose

v0.9.11 is not only about preference and fit.

It must also prepare the project structure for the next two releases:

```text
v0.9.12:
  build a few standard KisteUnits bundled with Kiste Core as a standard library

v0.9.13:
  split Kiste into five clean repositories
```

Therefore v0.9.11 should add boilerplate for:

```text
standard KisteUnit layout
strong Go hook foundation
tool integration unit layout
workspace compatibility layout
future five-repo split
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
Make five-repo split predictable.
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
Split Kiste into five repositories.
Preserve the same package boundaries.
Preserve the same import rules.
Preserve the same unit contracts.
Keep the existing kiste-spec repository as the spec anchor.
```

---

## 3. Correct Five Repositories

The future five repositories are:

```text
1. kiste-cli
2. kiste-py
3. kiste-unit-sdk
4. kiste-core
5. kiste-spec
```

Important correction:

```text
kiste-spec is already a separate repository.

The split is not four repos.
The correct long-term split is five repos.
```

Meaning:

```text
kiste-cli:
  CLI command surface, terminal UX, command routing, and user-facing executable package

kiste-py:
  Python SDK facade, programmatic API, Python package surface, and developer import path

kiste-unit-sdk:
  builders, Tool Weaver, integration-unit scaffolding, KisteUnit test harnesses, package helpers

kiste-core:
  Core Engine, lifecycle engine, capability graph, hook runtime, policy engine, plan/review/monitor models

kiste-spec:
  existing spec repository, release specs, schemas, capability contracts, reference architecture, standard-unit specs
```

Standard KisteUnits rule:

```text
There is no separate kiste-standard-units repository in the initial split.

Standard KisteUnits are specified in kiste-spec and bundled/released with kiste-core as the Kiste standard library.

They may later be extracted if needed, but they are not one of the first five repositories.
```

Optional later repositories:

```text
kiste-standard-units, only if the standard library becomes too large later
kiste-go-hooks
kiste-manager
kiste-cloud
kiste-examples
```

---

## 4. Repositories May Live Together First

Before the split, these five repos may live together in one development workspace.

Recommended temporary layout:

```text
kiste-workspace/
  repos/
    kiste-cli/
    kiste-py/
    kiste-unit-sdk/
    kiste-core/
    kiste-spec/
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
The code may live together during development, but boundaries must already behave like five separate repositories.

The existing kiste-spec repo remains the specification anchor.
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
  name: kiste-core

spec:
  future_repo: github.com/KisteBox/kiste-core
  current_path: core/

  owns:
    - lifecycle
    - capability_graph
    - hook_runtime
    - policy_engine
    - plan_model
    - review_model
    - monitor_model

  must_not_import:
    - kiste-cli
    - kiste-py
    - kiste-unit-sdk
    - kiste-spec-internals
```

Spec repo boundary:

```yaml
apiVersion: kiste.dev/v0.9.11
kind: KisteRepoBoundary

metadata:
  name: kiste-spec

spec:
  future_repo: github.com/KisteBox/kiste-spec
  current_path: spec/
  already_exists: true

  owns:
    - release_specs
    - schemas
    - capability_contracts
    - reference_architecture
    - standard_unit_specs

  must_not_import_runtime_code: true
```

---

## 6. Import Rules for Future Split

Allowed:

```text
kiste-cli -> kiste-py
kiste-cli -> kiste-core only through stable CLI-facing APIs if needed
kiste-py -> kiste-core
kiste-unit-sdk -> kiste-core contracts
kiste-core -> kiste-spec schemas/contracts at build/test/spec-validation time
standard bundled units -> kiste-core contracts
standard bundled units -> kiste-spec definitions
```

Forbidden:

```text
kiste-core -> kiste-cli
kiste-core -> kiste-py
kiste-core -> kiste-unit-sdk internals
kiste-spec -> runtime code
kiste-spec -> kiste-cli
kiste-spec -> kiste-py
kiste-spec -> kiste-unit-sdk
kiste-cli -> kiste-unit-sdk internals
kiste-py -> kiste-unit-sdk internals
standard bundled units -> kiste-cli internals
standard bundled units -> kiste-py internals
```

Rule:

```text
Core has no upward runtime dependencies.
Spec has no runtime dependencies.
CLI is thin.
Python SDK is the public programmatic facade.
Unit SDK is for building units, not for running Core.
Standard bundled units depend on Core contracts and Spec definitions, not Core internals.
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
Standard KisteUnits are specified in kiste-spec and bundled/released with kiste-core.
They must remain replaceable by external KisteUnits.
```

---

## 11. Standard Library Model

Standard KisteUnits are like a language standard library.

They provide common behavior without forcing users to install everything.

Characteristics:

```text
specified in kiste-spec
bundled with kiste-core
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
    specified_in: kiste-spec
    bundled_in: kiste-core
    bundled: true
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
2. Standard bundled KisteUnit from kiste-core
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
.kiste/repo-boundaries/five-repo-boundary-report.json
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
1. The correct five future repositories are named.
2. The five repositories are kiste-cli, kiste-py, kiste-unit-sdk, kiste-core, and kiste-spec.
3. kiste-spec is recognized as already separate and existing.
4. kiste-core is separated from kiste-spec in the future split.
5. There is no separate kiste-standard-units repo in the initial five-repo split.
6. Repo boundary markers are defined.
7. Import rules for the future split are defined.
8. Monorepo development remains possible.
9. Multi-repo split is mechanically prepared.
10. Strong Go hook interface is defined.
11. Go-native tool adapters are named.
12. Standard KisteUnit catalog is prepared.
13. v0.9.12 standard-unit scope is defined.
14. v0.9.13 five-repo split scope is defined.
15. Standard units are replaceable by external units.
16. Core does not import CLI, Python SDK, or Unit SDK internals.
17. Spec does not import runtime code.
```

---

## 16. Final Rule

```text
v0.9.11 prepares the boilerplate.

v0.9.12 builds the standard KisteUnit library specified in kiste-spec and bundled with kiste-core.

v0.9.13 splits the repositories five ways:

  1. kiste-cli
  2. kiste-py
  3. kiste-unit-sdk
  4. kiste-core
  5. kiste-spec

kiste-spec already exists as a separate spec repository.

The five repos may live together during development, but the boundaries must already behave like separate repositories.

Strong Go hooks are prepared in v0.9.11 because Kubernetes, Docker, OCI, Kubeflow, and many CNCF tools are Go-native.
```
