# Kiste v0.9.10 — Replace Provider Model with Tool, Capability, KisteUnit, and Workspace Integration Model

Status: Architecture correction  
Release: `0.9.10`  
Theme: Provider is removed as a first-class model and replaced by tools, capabilities, KisteUnits, and workspace integration.

---

## 1. Correction

Kiste should not keep `Provider` as a first-class model.

The corrected model is:

```text
Capability is the contract.
Tool is the external implementation surface.
KisteUnit is the integration package.
Workspace is the binding context.
```

Therefore:

```text
Provider model is replaced by:
  tool model
  capability model
  KisteUnit integration model
  workspace binding model
```

Final rule:

```text
Kiste speaks capabilities.
Kiste integrates tools through KisteUnits.
Workspace decides which capability implementation is allowed.
```

---

## 2. Why Provider Should Be Removed

The word `provider` is too broad.

It can mean:

```text
cloud provider
runtime provider
Git provider
secret provider
tool provider
API provider
plugin provider
managed service provider
```

This causes abstraction confusion.

Kiste should instead use more precise concepts:

```text
Capability = what Kiste needs
Tool = existing external system or API
KisteUnit = integration package
Workspace = project-specific binding and policy
ResolvedCapability = selected implementation for one capability
```

---

## 3. New Vocabulary

### Capability

A capability is what Kiste needs or can perform.

Examples:

```text
runtime.kubernetes
git.update
iam.action_plan
container.oci_metadata
gitops.sync_status
progressive.rollout
monitor.runtime_health
```

### Tool

A tool is an existing system, SDK, API, CLI, framework, or platform that can implement capabilities.

Examples:

```text
Kubernetes
Docker
Docker Compose
OCI registry
Helm
Kustomize
Argo CD
Flux
GitHub
GitLab
boto3
AWS SDK
Prometheus
OpenTelemetry
SOPS
Vault
```

### KisteUnit

A KisteUnit packages the integration between Kiste and a tool or group of tools.

Examples:

```text
kiste-unit-kubernetes
kiste-unit-docker-compose
kiste-unit-oci
kiste-unit-github
kiste-unit-iam-manager
kiste-unit-prometheus
```

### Workspace Binding

The workspace decides which implementation is allowed for a capability.

It applies:

```text
policy
trust
region
account boundary
secret boundary
mutation boundary
approval requirement
unit lock
tool availability
```

---

## 4. Model Replacement

Remove this model:

```yaml
providers:
  - name: kubernetes
    type: runtime-provider
    provides:
      capabilities:
        - runtime.kubernetes
```

Replace with this model:

```yaml
spec:
  requires:
    capabilities:
      - runtime.kubernetes
      - runtime.manifest_validation
      - runtime.health

  tools:
    available:
      - name: kubernetes
        kind: tool
        access: existing-api
      - name: docker-compose
        kind: tool
        access: file-format

  units:
    requires:
      - module: github.com/KisteBox/kiste-unit-kubernetes
        version: v0.2.0
      - module: github.com/KisteBox/kiste-unit-docker-compose
        version: v0.1.0

  capability_resolution:
    runtime.kubernetes:
      allowed_implementations:
        - unit: github.com/KisteBox/kiste-unit-kubernetes
          tool: kubernetes
          mode: existing-api-adapter
```

---

## 5. Capability Owns Implementation Candidates

A capability may have many possible implementations.

Example:

```yaml
apiVersion: kiste.dev/v0.9.10
kind: Capability

metadata:
  name: runtime.kubernetes

spec:
  family: runtime

  requires:
    - policy.validate
    - runtime.manifest_validation
    - secret.ref

  implementations:
    candidates:
      - name: kubernetes-basic-core-hook
        source: core-hook
        tools:
          - kubernetes-manifest

      - name: kiste-unit-kubernetes
        source: kiste-unit
        module: github.com/KisteBox/kiste-unit-kubernetes
        tools:
          - kubernetes-api

      - name: gitops-argocd-integration
        source: kiste-unit
        module: github.com/KisteBox/kiste-unit-argocd
        tools:
          - argocd-api
          - kubernetes-api
```

This replaces `ProviderGraph`.

---

## 6. Tool Model

A tool is not the same thing as a capability.

A tool may implement many capabilities.

A capability may be implemented by many tools.

Example:

```yaml
apiVersion: kiste.dev/v0.9.10
kind: Tool

metadata:
  name: kubernetes

spec:
  category: runtime
  interface:
    type: existing-api
    language: go
    sdk_compatible: true

  can_implement:
    capabilities:
      - runtime.kubernetes
      - runtime.manifest_validation
      - runtime.dry_run
      - runtime.health
      - runtime.drift
      - rollout.status
```

Tool rule:

```text
Tools do not enter Kiste directly.
Tools enter Kiste through Core hooks or KisteUnits.
```

---

## 7. KisteUnit Integration Model

A KisteUnit binds one or more tools to Kiste capabilities.

Example:

```yaml
apiVersion: kiste.dev/v0.9.10
kind: KisteUnit

metadata:
  name: kubernetes-integration-unit
  module: github.com/KisteBox/kiste-unit-kubernetes
  version: v0.2.0

spec:
  category: existing-tool-integration

  integrates:
    tools:
      - kubernetes

  provides:
    capabilities:
      - runtime.kubernetes
      - runtime.manifest_validation
      - runtime.dry_run
      - runtime.health
      - runtime.drift

  requires:
    capabilities:
      - git.update
      - policy.validate
      - secret.ref

  policy:
    no_mutation_during_read: true
    no_mutation_during_inspect: true
    approved_plan_required: true
```

---

## 8. Workspace Integration Model

Workspace binds capabilities to KisteUnits/tools under policy.

Example:

```yaml
apiVersion: kiste.dev/v0.9.10
kind: Workspace

metadata:
  name: ai-product-platform

spec:
  requires:
    capabilities:
      - workspace.read
      - repo.discover
      - git.state_read
      - runtime.kubernetes
      - runtime.manifest_validation
      - gitops.manifest_export
      - iam.action_plan

  tools:
    allowed:
      - kubernetes
      - docker
      - docker-compose
      - oci-registry
      - github

  units:
    requires:
      - module: github.com/KisteBox/kiste-unit-kubernetes
        version: v0.2.0
      - module: github.com/KisteBox/kiste-unit-iam-manager
        version: v0.1.0

  capability_resolution:
    strategy: safest-allowed

    bindings:
      runtime.kubernetes:
        unit: github.com/KisteBox/kiste-unit-kubernetes
        tool: kubernetes
        mode: existing-api-adapter

      iam.action_plan:
        unit: github.com/KisteBox/kiste-unit-iam-manager
        tool: iam-api
        mode: action-plan

    policy:
      require_lock_file_for_units: true
      require_approved_plan_for_mutation: true
      prefer_core_for_basic_plans: true
      prefer_unit_for_deep_tool_behavior: true
```

---

## 9. Graph Model

Replace:

```text
ProviderGraph
```

With:

```text
CapabilityDependencyGraph
CapabilityImplementationGraph
ToolIntegrationGraph
ResolvedCapabilityGraph
WorkspaceBindingGraph
```

Meaning:

```text
CapabilityDependencyGraph:
  what capabilities depend on other capabilities

CapabilityImplementationGraph:
  what implementation candidates can satisfy each capability

ToolIntegrationGraph:
  what tools are integrated by which KisteUnits/hooks

ResolvedCapabilityGraph:
  which implementation was selected for each capability

WorkspaceBindingGraph:
  what this workspace allows and selected
```

---

## 10. Resolution Flow

```text
1. Read workspace required capabilities.
2. Build global capability dependency graph.
3. Find tools available to the workspace.
4. Find KisteUnits that integrate those tools.
5. Build capability implementation graph.
6. Apply workspace policy.
7. Apply unit lock/trust policy.
8. Apply mutation boundary.
9. Select implementation for each capability.
10. Emit ResolvedCapabilityGraph.
```

---

## 11. Output Layout

```text
.kiste/capabilities/
  global-capability-dependency-graph.json
  capability-implementation-graph.json
  resolved-capability-graph.json
  missing-capability-report.json

.kiste/tools/
  available-tools.json
  tool-integration-graph.json
  tool-compatibility-report.json

.kiste/units/
  required-units.json
  resolved-units.json
  unit-lock-report.json

.kiste/workspace/
  workspace-binding-graph.json
```

---

## 12. Acceptance Criteria

This correction is accepted only if:

```text
1. Provider is removed as a first-class model.
2. Workspace declares required capabilities, not required providers.
3. Tools are modeled as existing implementation surfaces.
4. KisteUnits integrate tools with Kiste capabilities.
5. CapabilityImplementationGraph replaces ProviderGraph.
6. ToolIntegrationGraph records tool-to-unit relationships.
7. WorkspaceBindingGraph records workspace-specific selections.
8. ResolvedCapability records selected unit/tool/hook implementation.
9. Kiste review blocks plans with unresolved required capabilities.
10. The control plane speaks capabilities, not providers.
```

---

## 13. Final Rule

```text
Provider is removed.

Kiste uses:

  Capability = what is needed
  Tool = existing implementation surface
  KisteUnit = integration package
  Workspace = policy-bound context
  ResolvedCapability = selected implementation

Kiste does not resolve providers.
Kiste resolves capabilities to tool-backed KisteUnit implementations inside a workspace.
```
