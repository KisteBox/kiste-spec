# Kiste v0.9.12 — Canonical YAML and Capability Model

Status: Draft canonical standard  
Release: `0.9.12`  
API: `kiste.dev/v1alpha1`  
Theme: One strict YAML envelope, a small capability contract, deterministic resolution, and explicit separation between user intent and generated state.

---

## 1. Decision

Kiste v0.9.12 standardizes four objects:

```text
Workspace   user intent, sources, policy, requirements, and explicit bindings
KisteUnit   an implementation package that integrates tools through one adapter
Capability  a stable behavioral contract and its dependencies
KisteLock   generated, immutable resolution for one workspace state
```

All objects use the same envelope:

```yaml
apiVersion: kiste.dev/v1alpha1
kind: Workspace
metadata:
  name: example
spec: {}
```

`apiVersion` is the schema version, not the Kiste product release. Kiste `0.9.12`, `0.9.13`, and later releases may all read `kiste.dev/v1alpha1` until the API contract changes.

---

## 2. Source-of-Truth Files

| File | Owner | Purpose |
| --- | --- | --- |
| `kiste.yaml` | User | Workspace intent and policy |
| `kiste.unit.yaml` | Unit author | Unit contract and adapter declaration |
| `capabilities/*.yaml` | Spec or extension author | Capability definitions |
| `kiste.lock.yaml` | Kiste | Exact resolved units and capability implementations |
| `.kiste/**` | Kiste | Reports, graphs, evidence, plans, and runtime observations |

Rules:

```text
User intent belongs in kiste.yaml.
Implementation metadata belongs in kiste.unit.yaml.
Capability meaning belongs in capability definitions.
Selection results belong in kiste.lock.yaml.
Observed status never belongs in authored manifests.
```

`kiste.lock.yaml` and `.kiste/**` must not contain secret values.

---

## 3. Vocabulary

```text
Capability = what behavior is needed
Tool       = an external API, SDK, CLI, file format, framework, or platform
KisteUnit  = the package that maps one or more tools to capabilities
Adapter    = how Core invokes a KisteUnit
Workspace  = the policy-bound context in which resolution occurs
Binding    = an explicit workspace choice for one capability
Resolution = the frozen capability-to-unit selection
```

`Provider` remains removed as a first-class Kiste object. A cloud company, Git service, or runtime may be named as a tool or source, but it does not create a generic provider abstraction.

---

## 4. YAML Conventions

All standard manifests must follow these rules:

1. UTF-8 YAML 1.2.
2. One Kiste object per file. Multi-document YAML is not part of `v1alpha1`.
3. Unknown fields fail validation.
4. Field names use `lowerCamelCase`.
5. Enum values and identifiers use lowercase kebab-case unless a contract says otherwise.
6. Collections whose order has no semantic meaning are sorted before hashing.
7. Duplicate keys are invalid.
8. Anchors and aliases may be rejected by security-sensitive loaders.
9. Secret values are forbidden; manifests contain references only.
10. Generated fields are not accepted in authored objects.

The machine-readable contract is `schemas/kiste-v1alpha1.schema.json`.

---

## 5. Common Envelope

Required fields:

```yaml
apiVersion: kiste.dev/v1alpha1
kind: Workspace | KisteUnit | Capability | KisteLock
metadata:
  name: lowercase-name
spec: {}
```

Optional metadata:

```yaml
metadata:
  name: ai-product-platform
  labels:
    team: platform
  annotations:
    docs.kiste.dev/owner: platform-team
```

Workspace, KisteUnit, and KisteLock names must be DNS-label compatible. Capability names use the dotted identifier defined below. Labels are used for selection. Annotations are non-semantic metadata and must not change resolution.

---

## 6. Canonical Workspace

`kiste.yaml` has `kind: Workspace`.

```yaml
apiVersion: kiste.dev/v1alpha1
kind: Workspace

metadata:
  name: ai-product-platform

spec:
  intent:
    outcome: secure.application_keys
    subject: application

    context:
      cloud: azure
      runtime: aks
      identityBoundary: azure-tenant
      region: southeast-asia
      environment: production

    dealBreakers:
      - name: azure-key-boundary
        appliesTo:
          capabilities:
            - security.key_encryption
        must:
          cloudAffinity:
            - azure
            - cloud-neutral
          identityBoundaries:
            - azure-tenant
            - workload-identity
          externalCloudDependency: false

    preferences:
      order:
        - same-cloud
        - same-identity-boundary
        - managed-service
        - verified
        - fewest-permissions
        - fewest-units

  sources:
    - name: application
      type: git
      uri: https://github.com/example/application.git
      ref: main

    - name: company-runbooks
      type: directory
      path: ./runbooks

    - name: release-artifacts
      type: object-store
      uri: s3://example-release-artifacts/releases/
      authRef: aws-release-reader

  capabilities:
    requires:
      - git.state_read
      - container.oci_metadata
      - runtime.manifest_validation
      - security.key_encryption
    optional:
      - monitor.runtime_health

  units:
    - module: std:git
      version: 0.9.12
    - module: std:oci
      version: 0.9.12
    - module: github.com/KisteBox/kiste-unit-kubernetes
      version: v0.2.0
    - module: github.com/KisteBox/kiste-unit-azure-key-vault
      version: v0.1.0

  resolution:
    mode: locked
    onAmbiguity: fail
    allowFallback: false
    bindings:
      - capability: runtime.manifest_validation
        unit: std:kubernetes-basic

  policy:
    allowedSourceHosts:
      - github.com
      - gitlab.com
      - bitbucket.org
    allowedAdapters:
      - native-go
      - local-grpc
      - wasm
    mutation:
      requireApprovedPlan: true
      readOnlyStages:
        - read
        - inspect
        - plan
        - review
    secrets:
      valuesInManifest: forbidden
```

### Source types

```text
git           GitHub, GitLab, Bitbucket, self-hosted Git, or another Git-compatible host
directory     a local workspace directory
object-store  S3, GCS, Azure Blob, MinIO, or another object-store URI
oci           an OCI artifact or registry reference
http          a read-only HTTP(S) artifact
```

Kiste identifies source behavior from `type` and URI. It must not require vendor-specific top-level fields. Vendor-specific options belong under a unit's typed configuration, not the common workspace schema.

### Credentials

`authRef` names a credential reference resolved by policy. It never contains a credential value. Environment variables, workload identity, secret managers, and OS keyrings may implement the reference.

---

## 7. Canonical Capability Identifier

A capability identifier is a lowercase, dot-separated name:

```text
family.action
family.subject.action
family.subject_qualifier.action
```

Examples:

```text
workspace.read
git.state_read
git.update
runtime.manifest_validation
iam.key.encryption_ref
monitor.runtime_health
```

Canonical pattern:

```regex
^[a-z][a-z0-9_]*(\.[a-z][a-z0-9_]*)+$
```

Rules:

1. The first segment is the capability family.
2. A capability describes behavior, not a vendor or product.
3. A capability is atomic enough to test with a conformance test.
4. Names are stable once published as `stable`.
5. Renames require a deprecation alias and migration period.
6. A unit may not invent a stable capability outside an allowed capability catalog.

Avoid:

```text
aws                         too broad and vendor-shaped
kubernetes                  a tool, not a capability
deploy.everything           not atomic
fast                        not testable
best.security               subjective
```

---

## 8. Canonical Capability Object

```yaml
apiVersion: kiste.dev/v1alpha1
kind: Capability

metadata:
  name: git.state_read

spec:
  family: git
  summary: Read repository state without mutation.
  stability: stable

  lifecycle:
    stages:
      - read
      - inspect
    mutation: none
    approval: never

  dependencies:
    requires:
      - workspace.read
    optional: []

  contract:
    inputs:
      - kiste.workspace-context/v1alpha1
    outputs:
      - kiste.git-facts/v1alpha1
    evidence:
      - source-revision
```

A Capability object defines meaning, dependencies, lifecycle boundary, and contract types. It does not list implementation candidates.

This corrects the earlier v0.9.10 candidate-in-capability shape:

```text
Capability definitions remain stable.
KisteUnits declare what they implement.
The resolver derives the CapabilityImplementationGraph.
```

That separation prevents capability contracts from changing whenever a new unit or tool appears.

---

## 9. Capability Stability

```text
experimental  shape may change without migration support
preview       shape is testable but may still change
stable        backward-compatible contract is required
deprecated    accepted for migration but not for new manifests
```

Capability stability describes the contract. It does not claim that every implementation is production-supported.

Implementation support is declared separately by a KisteUnit:

```text
experimental  implementation is incomplete or exploratory
compatible    passes schema and basic contract validation
verified      passes the published conformance suite
supported     has an identified maintainer and support policy
deprecated    accepted only for migration
```

---

## 10. Canonical KisteUnit

```yaml
apiVersion: kiste.dev/v1alpha1
kind: KisteUnit

metadata:
  name: github-basic

spec:
  module: github.com/KisteBox/kiste-unit-github
  version: v0.1.0

  adapter:
    type: local-grpc
    contract: kiste.hook/v1alpha1
    entrypoint: ./bin/kiste-unit-github

  tools:
    - name: github
      interface: api

  capabilities:
    provides:
      - name: git.state_read
        implementation: github-api
        support: verified
        traits:
          cloudAffinity:
            - cloud-neutral
          identityBoundaries:
            - external-saas
          runtimes:
            - any
          externalDependencies:
            - github-api
          externalCloudDependency: false
          managed: true
          portability: high
          operationalOverhead: low
      - name: git.update
        implementation: github-api
        support: compatible
    requires:
      - policy.validate
      - secret.ref

  lifecycle:
    stages:
      - read
      - inspect
      - plan
      - deploy
      - monitor

  policy:
    mutation: workspace
    requireApprovedPlan: true
    secretAccess: references-only
```

### Adapter types

| Adapter | Use | Required locator |
| --- | --- | --- |
| `native-go` | Trusted, in-process Go integration | `entrypoint` |
| `local-grpc` | Isolated local process | `entrypoint` |
| `remote-grpc` | Network service across an explicit trust boundary | `endpoint` |
| `wasm` | Sandboxed portable module | `module` and `digest` |

All adapters expose the same lifecycle contract. Adapter type must not change capability meaning.

`remote-grpc` is denied unless workspace policy explicitly allows it. Plaintext remote endpoints are invalid; the runtime must require authenticated transport and a declared trust policy.

---

## 11. Capability Ownership and Implementation

The responsibility boundary is:

```text
Capability catalog owns names and behavioral contracts.
KisteUnit owns the claim that it implements a capability.
Conformance tests verify the claim.
Workspace policy decides whether the unit is allowed.
KisteLock records what was selected.
```

One tool may support many capabilities. One capability may have many implementations. Kiste does not load all of them.

The active set is only:

```text
transitive closure(workspace required capabilities)
+ explicitly selected optional capabilities
+ lifecycle kernel capabilities
```

This is the minimum-capability principle.

---

## 12. Deterministic Resolution

The canonical YAML remains capability-oriented:

```text
Workspace spec.capabilities.requires = Need records
KisteUnit spec.capabilities.provides  = Offer records
KisteLock spec.capabilities           = Match records
```

`Need`, `Offer`, and `Match` are normalized resolver records, not additional manifest kinds. This keeps existing `kiste.yaml` and `kiste.unit.yaml` shapes compatible while allowing a much smaller matching engine.

Resolution order:

```text
1. Validate the Workspace and referenced manifests.
2. Read optional user intent, environment context, deal breakers, and preferences.
3. Expand required capability dependencies into Need records.
4. Normalize unit capability declarations into Offer records with compatibility traits.
5. Reject offers that fail capability, version, platform, tool, adapter, trust, or organization policy.
6. Reject offers that fail any applicable user deal breaker.
7. Apply explicit workspace bindings; an invalid binding fails rather than bypassing a gate.
8. Rank eligible offers using capability-specific preferences and then intent preferences.
9. Select the smallest complete unit set with the fewest permissions.
10. Fail on an unresolved need or ambiguous best candidate.
11. Write the exact matches and reasons to kiste.lock.yaml.
12. Build plans only from the locked matches.
```

Hard priority:

```text
technical and contract compatibility
organization policy and trust
applicable user deal breakers
explicit binding
capability-specific preference
intent-level preference
fewest units and permissions
fail on a remaining tie
```

Policy or a deal breaker can block a preferred implementation. No numeric score may compensate for a failed hard gate. A tie is an ambiguity, not permission to make a hidden vendor choice.

### Resolution modes

```text
locked  an existing valid lock is required; no automatic change
update  Kiste may update candidates and produces a lock-file diff for review
```

### Ambiguity behavior

`onAmbiguity: fail` is the only `v1alpha1` behavior. The field is explicit so a later API can add reviewed alternatives without changing old manifests.

---

## 13. User Intent, Deal Breakers, and Preferences

User intent is optional. A capability-only Workspace remains valid and resolves exactly as before.

Intent contributes three matching inputs:

```text
context        facts about the target environment
dealBreakers   scoped hard constraints that eliminate offers
preferences    ordered soft criteria for eligible offers
```

Deal breakers must be scoped to capability names or families. A rule for key management must not accidentally reject an unrelated Git or monitoring unit.

```yaml
intent:
  outcome: secure.application_keys
  subject: application

  context:
    cloud: azure
    runtime: aks
    identityBoundary: azure-tenant
    region: southeast-asia
    environment: production

  dealBreakers:
    - name: azure-key-boundary
      appliesTo:
        capabilities:
          - security.key_encryption
      must:
        cloudAffinity:
          - azure
          - cloud-neutral
        identityBoundaries:
          - azure-tenant
          - workload-identity
        externalCloudDependency: false

  preferences:
    order:
      - same-cloud
      - same-identity-boundary
      - managed-service
      - verified
      - fewest-permissions
      - fewest-units
```

KisteUnits advertise matching facts on each capability offer:

```yaml
capabilities:
  provides:
    - name: security.key_encryption
      implementation: azure-key-vault
      support: verified
      traits:
        cloudAffinity:
          - azure
        identityBoundaries:
          - azure-tenant
          - workload-identity
        runtimes:
          - aks
        externalDependencies:
          - azure-key-vault-api
        externalCloudDependency: false
        managed: true
        portability: low
        operationalOverhead: low
```

### AWS KMS and Azure example

An AWS KMS offer may correctly provide `security.key_encryption`, but advertise:

```yaml
traits:
  cloudAffinity:
    - aws
  identityBoundaries:
    - aws-account
  externalDependencies:
    - aws-kms-api
  externalCloudDependency: true
```

For the Azure intent above, the matcher rejects it:

```yaml
candidate: aws-kms
status: rejected
reasons:
  - cloud-affinity-not-allowed
  - identity-boundary-mismatch
  - external-cloud-dependency-forbidden
```

This is not a universal rule that AWS KMS can never be called from Azure. If another workspace explicitly allows cross-cloud dependencies, the offer may remain eligible. Compatibility is derived from user intent and policy, not hardcoded vendor prejudice.

### Capability-specific preferences

Existing `resolution.preferences` remain valid. They rank eligible candidates for a specific capability after all hard gates pass.

```yaml
resolution:
  mode: update
  onAmbiguity: fail
  allowFallback: true
  preferences:
    - capability: mlops.pipeline
      when:
        workloadType: ml-pipeline
        runtime: kubernetes
      prefer:
        unit: github.com/KisteBox/kiste-unit-kubeflow
      reason: native-ml-pipeline-support
```

Preferences must be local, ordered, explainable, and reviewable. Unit authors advertise facts; they cannot assign themselves a workspace fit score. If an offer omits a trait required by an applicable deal breaker, the matcher rejects it as `required-trait-unknown`.

---

## 14. Generated KisteLock

```yaml
apiVersion: kiste.dev/v1alpha1
kind: KisteLock

metadata:
  name: ai-product-platform

spec:
  workspaceDigest: sha256:1111111111111111111111111111111111111111111111111111111111111111
  intentDigest: sha256:3333333333333333333333333333333333333333333333333333333333333333

  units:
    - module: github.com/KisteBox/kiste-unit-kubernetes
      version: v0.2.0
      commit: 8f3ab21
      digest: sha256:2222222222222222222222222222222222222222222222222222222222222222
    - module: github.com/KisteBox/kiste-unit-azure-key-vault
      version: v0.1.0
      commit: 9a4bc32
      digest: sha256:4444444444444444444444444444444444444444444444444444444444444444

  capabilities:
    - name: runtime.manifest_validation
      unit: github.com/KisteBox/kiste-unit-kubernetes
      implementation: kubernetes-api
      adapter: native-go
      support: verified
      reasons:
        - capability-compatible
        - policy-allowed

    - name: security.key_encryption
      unit: github.com/KisteBox/kiste-unit-azure-key-vault
      implementation: azure-key-vault
      adapter: local-grpc
      tool: azure-key-vault
      support: verified
      reasons:
        - capability-compatible
        - deal-breakers-passed
        - same-cloud
        - same-identity-boundary
        - managed-service

  generatedBy:
    kisteVersion: 0.9.12
    schemaVersion: kiste.dev/v1alpha1
```

The lock is generated and deterministic. A change to intent, deal breakers, units, versions, digests, bindings, capability selections, or match reasons must appear as a reviewable lock diff. Rejected candidates belong in `.kiste/capabilities/implementation-selection-report.json`; the lock stores only selected matches and concise reasons.

---

## 15. Lifecycle and Mutation

The six lifecycle stages remain:

```text
read -> inspect -> plan -> review -> deploy -> monitor
```

Mutation levels:

```text
none             facts, validation, reports, and evidence only
workspace        approved changes to workspace files or Git
runtime          approved changes to a workload runtime
external-system  approved changes to cloud, SaaS, IAM, or another remote control plane
```

Rules:

```text
Read and inspect never mutate.
Plan describes changes but does not apply them.
Review records decisions but does not apply changes.
Deploy may mutate only through an approved locked plan.
Monitor is read-only unless it creates a new plan for review.
```

---

## 16. Validation Invariants

JSON Schema validates structure. Kiste must additionally enforce:

1. `Capability.spec.family` equals the first identifier segment.
2. Every required capability exists and is not deprecated for new use.
3. Capability dependencies are acyclic unless an explicit fixed-point contract exists in a later API.
4. A unit's provided capability exists in an allowed catalog.
5. A unit adapter is allowed by workspace policy.
6. A `verified` claim has matching conformance evidence.
7. `supported` includes maintainer and support metadata in the unit release record.
8. Mutation declared by a unit is not weaker than any capability it provides.
9. Remote gRPC uses authenticated transport and an explicit trust policy.
10. No manifest or lock contains a secret value.
11. Required units are version-pinned; production locks also require immutable digests.
12. Resolution activates only the minimum required capability closure.
13. A Workspace without `spec.intent` remains valid and uses capability-only resolution.
14. Every deal breaker is scoped to one or more capability names or families.
15. An offer missing a trait required by an applicable deal breaker is rejected as unknown, not treated as compatible.
16. A failed deal breaker cannot be offset by preference weight or optimizer score.
17. Unit-advertised traits are facts subject to validation; units cannot assign themselves preference scores.
18. The lock records concise reasons for every selected match.

---

## 17. Migration from v0.9.x Shapes

| Legacy shape | `v1alpha1` replacement |
| --- | --- |
| `kiste: "0.9.x"` plus `schema:` | `apiVersion: kiste.dev/v1alpha1` |
| Release-numbered API such as `kiste.dev/v0.9.11` | Release-independent `kiste.dev/v1alpha1` |
| `providers:` | Tools integrated by KisteUnits |
| Capability object containing candidates | Unit `capabilities.provides` plus derived graph |
| Free-form capability names | Cataloged dotted identifiers |
| Resolution result in `kiste.yaml` | Generated `kiste.lock.yaml` |
| Secret values in configuration | `authRef` or secret reference |
| Automatic tie-breaking | Explicit failure and binding |

Readers may support legacy manifests through a migration layer. Writers must emit only the canonical shape.

---

## 18. Compatibility Rules

1. Adding an optional field is backward-compatible within `v1alpha1` only when old readers safely ignore it; because unknown fields are rejected, writers must negotiate supported schema versions first.
2. Removing or changing a required field requires a new API version.
3. Capability stable-contract changes require a new capability name or contract version.
4. Unit versioning follows the unit repository's semantic versioning.
5. Product release version and manifest API version evolve independently.

---

## 19. Required Artifacts

v0.9.12 publishes:

```text
Phase 9.12/kiste_v0_9_12_yaml_capability_standard.md
Phase 9.12/schemas/kiste-v1alpha1.schema.json
Phase 9.12/examples/kiste.yaml
Phase 9.12/examples/kiste.unit.yaml
Phase 9.12/examples/capability.git-state-read.yaml
Phase 9.12/examples/kiste.lock.yaml
```

Implementations must validate all examples against the published schema.

---

## 20. Acceptance Criteria

This standard is accepted only if:

1. Every manifest uses the common envelope.
2. API versioning is independent of product releases.
3. `kiste.yaml` has one canonical Workspace shape.
4. GitHub, GitLab, Bitbucket, object stores, and directories fit the common source model.
5. Capability identifiers follow the canonical pattern.
6. Capability definitions contain no implementation candidates.
7. KisteUnits declare capability implementations and adapter types.
8. Native Go, local gRPC, remote gRPC, and WASM adapters share one contract.
9. Workspace policy filters candidates before preference scoring.
10. Ambiguous or unresolved capabilities fail resolution.
11. Resolutions are frozen in `kiste.lock.yaml`.
12. Secret values are forbidden in manifests and locks.
13. Only the minimum required capability closure is active.
14. Unknown fields fail machine validation.
15. The published examples pass the published JSON Schema.
16. Existing capability-only Workspace manifests remain valid when `spec.intent` is absent.
17. Workspace intent supports context, scoped deal breakers, and ordered preferences.
18. KisteUnit capability offers may advertise typed compatibility traits.
19. Technical compatibility, organization policy, and user deal breakers filter candidates before preferences.
20. AWS KMS is rejected for an Azure-only key boundary but may remain eligible when cross-cloud dependencies are explicitly allowed.
21. The resolver uses derived Need, Offer, and Match records without introducing additional manifest kinds or mandatory graphs.

---

## 21. Final Rule

```text
Kiste YAML declares user intent, requirements, and policy, not discovered state.
Capability defines behavior, not implementation.
KisteUnit declares implementations and compatibility facts, not preference scores.
Workspace defines context, deal breakers, policy, bindings, and preferences.
KisteLock freezes selected matches and their reasons.

Kiste first filters incompatible offers, then ranks eligible offers, and finally activates only the smallest locked set required by the workspace.
```
