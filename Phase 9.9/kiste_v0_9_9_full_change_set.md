# Kiste v0.9.9 — Full Change Set

Status: Draft canonical release spec  
Release: `0.9.9`  
Theme: Core hook runtime, SDK tool builder, Tool Weaver, key management, Go-native tool integrations, existing API/SDK compatibility, and IAM management scaffold.

---

## 1. Release Theme

Kiste v0.9.9 moves the tool hook model into its proper architecture position.

The corrected direction is:

```text
Hook contract and hook runtime belong in Kiste Core.

Tool hook builder, Tool Weaver, adapter tests, and integration-unit scaffolding belong in Kiste SDK / KisteUnit SDK.

Deep tool behavior belongs in Existing Tool Integration Units.

IAM management enters as a privileged integration unit scaffold.

Key management is a special trust capability family.
```

One-sentence definition:

```text
Kiste v0.9.9 introduces the Core hook runtime, SDK Tool Weaver and builder APIs, Go-native tool integration units, existing API/SDK compatibility, key management as a special trust capability, and an IAM manager scaffold.
```

---

## 2. What Moves from v0.9.8 to v0.9.9

v0.9.8 remains focused on:

```text
SDK expansion
KisteUnit model
Kubernetes-lize foundation
hardware/security inspect
minimum operating capability kernel
four-way architecture split
```

v0.9.9 owns:

```text
Core hook runtime
tool hook contract
Tool Weaver SDK
integration-unit builder SDK
Go-native tool hooks
existing API/SDK compatibility
OCI/CNCF integration units
key management as special capability
IAM manager scaffold
```

---

## 3. Correct Architecture Placement

### Core owns hooks

```text
kiste_core:
  hook contract
  hook registry
  hook runtime
  hook policy gate
  lifecycle-safe invocation
```

Core must understand hooks because capability resolution, lifecycle execution, policy validation, and plan safety depend on them.

### SDK owns builders

```text
kiste_sdk:
  ToolHookBuilder
  ToolWeaver
  IntegrationUnitBuilder
  mock tool context
  adapter contract tests
  package/scaffold helpers
```

The SDK is the developer-facing layer for building new integrations.

### Integration Units own deep behavior

```text
KisteUnit / Existing Tool Integration Unit:
  existing tool API calls
  existing SDK usage
  deep behavior
  tool-specific normalization
  review evidence
  monitor signals
```

---

## 4. Updated Four-Way Architecture

v0.9.9 keeps the same four-way architecture:

```text
CLI
Python SDK
Core Engine
KisteUnit SDK
```

But clarifies the hook responsibility:

```text
Core Engine:
  runs Kiste
  owns hook runtime

Python SDK:
  exposes Kiste programmatically

KisteUnit SDK:
  builds KisteUnits
  builds integration units
  provides Tool Weaver and builder APIs

CLI:
  thin command surface
```

Final rule:

```text
Core runs hooks.
SDK builds hooks.
KisteUnits implement deep integrations.
```

---

## 5. Core Hook Runtime

Kiste Core must include a small, stable hook runtime.

Suggested package:

```text
kiste_core/
  hooks/
    models.py
    runtime.py
    registry.py
    policy.py
```

Core hook runtime responsibilities:

```text
define HookSpec
define HookContext
define HookResult
define lifecycle stages
register hooks
resolve hooks by capability
validate hook policy
block unsafe lifecycle use
invoke hook stage methods
normalize basic hook execution result
```

Core hook runtime must not:

```text
implement Kubernetes deeply
implement Docker deeply
implement OCI deeply
implement Helm deeply
implement Argo CD deeply
implement cloud SDKs deeply
implement IAM cloud mutation directly
```

---

## 6. Hook Contract

A hook is a Core-recognized bridge between Kiste and an external tool or tool family.

A hook must declare:

```text
tool name
tool family
implementation language when relevant
existing API compatibility
existing SDK compatibility
capabilities provided
capabilities required
lifecycle stages supported
files read
APIs called
facts produced
plans produced
reports produced
mutation policy
review evidence
monitor signals
```

Example:

```yaml
apiVersion: kiste.dev/v0.9.9
kind: KisteToolHook

metadata:
  name: docker-compose-hook

spec:
  tool:
    name: docker-compose
    family: container-orchestration
    implementation_language: go
    existing_api_compatible: true
    existing_sdk_compatible: true

  provides:
    capabilities:
      - compose.detect
      - compose.service_graph
      - compose.to_kubernetes_plan

  outputs:
    facts:
      - ComposeServiceGraph
    plans:
      - KubernetesRuntimePlan
      - PatchSet

  mutation:
    default: false
    requires_approved_plan: true

  policy:
    secret_values_allowed: false
```

---

## 7. Hook Execution Modes

Standard modes:

```text
observe-only
validate-only
render-only
render-and-apply
action-plan
hybrid
```

Default v0.9.9 modes:

```text
observe-only
validate-only
render-only
```

Mutating modes require an approved plan.

Rule:

```text
Hooks may observe, validate, or render without mutation.

Any hook that mutates Git, cluster, cloud, IAM, registry, CI/CD, or runtime must produce a reviewed and approved plan first.
```

---

## 8. SDK Tool Builder

The builder lives in the SDK.

Suggested package:

```text
kiste_sdk/
  builders/
    tool_hook_builder.py
    tool_weaver.py
    integration_unit_builder.py
```

SDK builder capabilities:

```text
sdk.tool_hook_build
sdk.integration_unit_build
sdk.existing_api_adapter
sdk.mock_tool_context
sdk.adapter_contract_test
sdk.integration_unit_package
sdk.integration_unit_validate
```

The SDK builder helps developers:

```text
scaffold a tool hook
declare tool metadata
declare capabilities
declare lifecycle handlers
declare existing API/SDK compatibility
declare mutation policy
map raw tool output to Kiste facts
map raw tool output to Kiste findings
map raw tool output to Kiste plans
map raw tool output to Kiste reports
test adapter contracts
package integration units
```

---

## 9. Tool Weaver

The previous Tool Weaver idea becomes the SDK builder layer.

Tool Weaver maps existing tool output into Kiste-native objects.

Mapping examples:

```text
kubectl / Kubernetes API output -> RuntimeFacts
Dockerfile / image metadata -> ContainerFacts
OCI registry metadata -> OCIImageFacts
Docker Compose model -> ComposeServiceGraph
Helm chart render -> ManifestRenderPlan
Argo CD app status -> GitOpsSyncStatus
Prometheus query -> ObservabilityMetricSignal
IAM policy document -> IAMFinding / IAMActionPlan
```

Tool Weaver outputs:

```text
facts
findings
plans
reports
evidence
monitor signals
```

Final rule:

```text
Tool Weaver belongs to the SDK because it is a developer tool for building integration units.
```

---

## 10. Existing Tool Integration Units

Deep integrations are packaged as KisteUnits.

Recommended term:

```text
Existing Tool Integration Unit
```

An integration unit wraps an existing tool through its API, SDK, CLI, file format, or object model.

It provides capabilities to Kiste without making Core depend on that tool.

Example:

```yaml
apiVersion: kiste.dev/v0.9.9
kind: KisteUnit

metadata:
  name: kubernetes-integration-unit
  module: github.com/KisteBox/kiste-unit-kubernetes
  version: v0.2.0

spec:
  category: existing-tool-integration

  tool:
    name: kubernetes
    implementation_language: go
    api_style: kubernetes-api
    sdk_compatibility:
      required: true

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

  hook:
    mode: existing-api-adapter
    mutation_default: false

  policy:
    no_mutation_during_read: true
    no_mutation_during_inspect: true
    approved_plan_required: true
    secret_values_allowed: false
```

---

## 11. Existing API and SDK Compatibility

v0.9.9 requires tool integrations to work with existing APIs and SDKs.

Rule:

```text
Kiste adapts to the tool's existing API/SDK.
Kiste does not force the tool to abandon its object model.
```

Examples:

```text
Kubernetes hook works with Kubernetes API concepts.
Docker/OCI hook works with image, digest, platform, registry, and metadata concepts.
Docker Compose hook works with service/network/volume/env-ref concepts.
Helm hook works with chart and values concepts.
Kustomize hook works with overlay concepts.
Argo CD hook works with application/sync/health concepts.
Prometheus hook works with query/metric concepts.
OpenTelemetry hook works with trace/log/metric signal concepts.
IAM hook works with identity, role, policy, permission boundary, and access analysis concepts.
```

---

## 12. Go-Native Tool Hook Family

The v0.9.9 hook model targets tools commonly written in Go and used in the cloud-native ecosystem.

Examples:

```text
Kubernetes
Docker / Moby / containerd ecosystem
Docker Compose
OCI container tooling
Helm
Kustomize
Argo CD
Flux
Prometheus
OpenTelemetry
Envoy / Gateway API
External Secrets
cert-manager
Kyverno
OPA / Gatekeeper
other CNCF tools
```

This does not mean Core imports all Go tools.

It means Core provides the hook runtime, while integration units use existing APIs/SDKs.

---

## 13. Kubernetes Hook

Kubernetes is the reference integration unit.

Capabilities:

```text
runtime.kubernetes
runtime.namespace
runtime.workload
runtime.service
runtime.config_ref
runtime.secret_ref
runtime.manifest_validation
runtime.dry_run
runtime.health
runtime.drift
runtime.rollout_status
runtime.events
```

Requirements:

```text
Must work with Kubernetes object model.
Must work with Kubernetes API behavior.
Must not become only a raw kubectl wrapper.
Must not mutate during read or inspect.
Must produce RuntimePlan, RuntimeFacts, review evidence, and monitor signals.
```

---

## 14. Docker and OCI Hook

Docker/OCI hooks focus on container identity, metadata, and runtime facts.

Capabilities:

```text
container.detect
container.dockerfile_read
container.image_ref
container.build_context
container.port_detect
container.env_detect
container.healthcheck_detect
container.security_hint
container.oci_metadata

oci.image_ref
oci.image_metadata
oci.digest
oci.platform
oci.architecture
oci.attestation_ref
oci.sbom_ref
oci.signature_ref
oci.registry_ref
```

Requirements:

```text
Must work with OCI image concepts.
Must support digest, platform, registry reference, and metadata concepts.
Must prefer digest/signature/attestation references over mutable tags for production.
```

---

## 15. Docker Compose Hook

Docker Compose hook helps Kiste understand local multi-service projects and plan migration to Kubernetes or another runtime.

Capabilities:

```text
compose.detect
compose.service_graph
compose.network_graph
compose.volume_graph
compose.env_ref
compose.secret_ref
compose.port_map
compose.to_kubernetes_plan
```

Rules:

```text
Compose hook may read .env references.
Compose hook must not expose .env values.
Compose hook must produce ComposeServiceGraph.
Compose hook may produce KubernetesMigrationHints and PatchSet.
```

---

## 16. CNCF Tool Hooks

CNCF tools should be integrated as hooks/KisteUnits, not core dependencies.

Examples:

```text
Helm hook
Kustomize hook
Argo CD hook
Flux hook
Prometheus hook
OpenTelemetry hook
Envoy/Gateway API hook
External Secrets hook
cert-manager hook
Kyverno/Gatekeeper policy hook
```

Possible capabilities:

```text
runtime.manifest_render
gitops.sync_status
gitops.drift
gitops.health
policy.admission_check
policy.runtime_check
secret.external_ref
observability.metrics_ref
observability.trace_ref
observability.logs_ref
```

---

## 17. Core Simple Docker and Kubernetes Support

Core may support simple built-in understanding of common formats.

Core may know:

```text
basic Dockerfile shape
basic Compose service graph
basic Kubernetes manifest shape
basic OCI image reference shape
```

Core may support:

```text
simple Dockerfile detection
simple OCI image metadata detection
simple docker-compose detection
simple Kubernetes manifest generation
simple Kubernetes manifest validation
simple resource policy suggestions
simple SecretRef checks
```

Core must not own:

```text
full Docker engine behavior
full Kubernetes client behavior
full Helm lifecycle
full Argo CD lifecycle
full Prometheus query engine
full OpenTelemetry collector behavior
full CNCF tool lifecycle
```

Deep behavior belongs to integration units.

---

## 18. Key Management as Special Capability

Key management is a special trust capability family in v0.9.9.

Capability families:

```text
key.*
secret.*
identity.*
signing.*
trust.*
```

Recommended capabilities:

```text
key.ref
key.identity
key.signing
key.verification
key.encryption_ref
key.rotation_policy
key.access_policy
key.audit

secret.ref
secret.backend_ref
secret.inject_ref
secret.rotation_policy
secret.access_policy
secret.no_plaintext

identity.workload
identity.service_account
identity.oidc
identity.federation
identity.role_assumption

signing.plan
signing.commit
signing.artifact
signing.unit
signing.attestation

trust.policy
trust.boundary
trust.manager_identity
trust.unit_source
trust.provider_identity
```

Hard rule:

```text
Kiste manages references and policies, not raw secret values.
```

Allowed:

```text
SecretRef
KeyRef
BackendRef
IdentityRef
SigningRef
TrustPolicyRef
```

Not allowed:

```text
secret_value
private_key_value
access_key_value
token_value
password_value
```

---

## 19. IAM Manager Scaffold

IAM management should enter as a privileged Existing Tool Integration Unit.

It is not baked directly into Core.

IAM manager capabilities:

```text
iam.read
iam.inspect
iam.policy_analyze
iam.least_privilege_check
iam.role_plan
iam.permission_boundary_check
iam.action_plan
iam.approved_apply
```

IAM manager requirements:

```text
No direct IAM mutation during read.
No direct IAM mutation during inspect.
No secret values in reports.
IAM changes must become ActionPlan entries.
IAM mutations require approved plan.
Least-privilege findings are review evidence.
Permission boundary checks are required for risky plans.
```

Example:

```yaml
apiVersion: kiste.dev/v0.9.9
kind: KisteUnit

metadata:
  name: iam-manager-integration-unit
  module: github.com/KisteBox/kiste-unit-iam-manager
  version: v0.1.0

spec:
  category: existing-tool-integration

  tool:
    name: iam-manager
    family: identity-access-management
    implementation_language: mixed
    existing_api_compatible: true
    existing_sdk_compatible: true

  provides:
    capabilities:
      - iam.read
      - iam.inspect
      - iam.policy_analyze
      - iam.least_privilege_check
      - iam.role_plan
      - iam.permission_boundary_check
      - iam.action_plan

  requires:
    capabilities:
      - key.ref
      - secret.ref
      - identity.oidc
      - trust.policy
      - policy.validate
      - review.validate

  hook:
    mode: action-plan
    mutation_default: false

  policy:
    no_mutation_during_read: true
    no_mutation_during_inspect: true
    approved_plan_required: true
    secret_values_allowed: false
    require_permission_boundary: true
    require_least_privilege_report: true
```

---

## 20. IAM Manager Code Scaffold

Suggested scaffold:

```text
examples/
  iam_manager_unit/
    README.md
    kiste_unit.yaml
    iam_manager.py
```

The scaffold should include:

```text
IAMManagerHook
IAMFacts
IAMLeastPrivilegeReport
IAMActionPlan
IAMReviewEvidence
IAMDriftSignal
```

Mutation rule:

```text
IAM deploy must refuse execution unless an approved IAMActionPlan is present.
```

---

## 21. Output Layout Additions

v0.9.9 adds:

```text
.kiste/
  hooks/
    hook-registry.json
    hook-resolution-report.json
    hook-policy-report.json

  sdk/
    tool-weaver-report.json
    adapter-contract-report.json
    integration-unit-build-report.json

  integration-units/
    resolved-units.json
    unit-contract-report.json
    unit-test-report.json

  iam/
    iam-facts.json
    iam-least-privilege-report.json
    iam-action-plan.json
    iam-review-evidence.json
    iam-drift.json

  key-management/
    key-ref-report.json
    secret-ref-report.json
    trust-policy-report.json
    signing-ref-report.json
```

---

## 22. v0.9.9 Non-Goals

v0.9.9 does not include:

```text
full IAM product
full cloud IAM mutation engine
full Kubernetes client implementation in Core
full Docker engine implementation in Core
full Helm implementation in Core
full Argo CD implementation in Core
full CNCF integration marketplace
automatic privileged IAM mutation
secret value storage in Kiste
direct secret exposure to KisteUnits
```

---

## 23. Acceptance Criteria

v0.9.9 is accepted only if:

```text
1. Core defines HookSpec, HookContext, HookResult, and lifecycle stages.
2. Core includes HookRegistry.
3. Core includes HookRuntime.
4. Core validates hook policy before invocation.
5. Core blocks mutation during read and inspect.
6. SDK includes ToolHookBuilder.
7. SDK includes ToolWeaver.
8. SDK includes IntegrationUnitBuilder.
9. Tool Weaver maps raw tool output to facts, findings, plans, reports, and evidence.
10. Existing Tool Integration Unit contract is defined.
11. Hook model explicitly targets Go-native platform tools.
12. Hook model supports Kubernetes, Docker, OCI, Docker Compose, and CNCF tools.
13. Hooks must work with existing tool APIs and SDKs.
14. Core remains independent from deep tool behavior.
15. Key management is documented as a special trust capability family.
16. Secret values are blocked from reports, plans, logs, and artifacts.
17. IAM manager scaffold is included.
18. IAM mutation requires approved ActionPlan.
19. IAM least-privilege report is part of review evidence.
20. Integration units can normalize tool output into Kiste facts/reports/plans.
```

---

## 24. Final Rule

```text
v0.9.9 establishes the hook layer correctly.

Hooks live in Kiste Core because Core must resolve, validate, and run hooks safely through the lifecycle.

The builder for hooks lives in the Kiste SDK.

The previous Tool Weaver becomes the SDK mapping layer for turning existing tool outputs into Kiste facts, findings, plans, reports, evidence, and monitor signals.

Deep tool behavior is packaged as Existing Tool Integration Units.

The target ecosystem is Go-native and cloud-native tooling: Kubernetes, Docker, OCI/container tooling, Docker Compose, and CNCF tools.

Hooks must work with existing APIs and SDKs.

Key management is a special trust capability family.

IAM management starts as a scaffolded privileged integration unit with approved-plan-only mutation.
```
