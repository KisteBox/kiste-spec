# Kiste v0.9.12 — SkyPilot Backend Integration

Status: Architecture proposal  
Release target: `0.9.12`  
Theme: Concede heterogeneous compute placement to SkyPilot and integrate it as a Kiste Existing Tool Integration Unit.

---

## 1. Decision

Kiste should not rebuild the compute-placement, accelerator catalog, cloud failover, or heterogeneous infrastructure scheduling work that SkyPilot already performs well.

The architectural decision is:

```text
Kiste owns:
  intent
  capabilities
  compatibility
  policy
  trust/evidence
  review/approval
  GitOps state
  execution contracts
  outcome records

SkyPilot owns:
  resource selection among allowed candidates
  cloud/region/zone placement
  Kubernetes/Slurm/cloud execution
  provisioning
  accelerator availability
  cost-aware placement
  failover
  managed-job recovery
```

SkyPilot is therefore a Tool integrated through a KisteUnit, not a Kiste core dependency and not a first-class Provider model.

---

## 2. Canonical Boundary

```text
User intent
    ↓
Kiste inspect / capability resolution
    ↓
Kiste compatibility + policy + evidence gates
    ↓
Approved Kiste ExecutionPlan
    ↓
Kiste SkyPilot Integration Unit
    ↓
SkyPilot Task / Resources
    ↓
SkyPilot placement and failover
    ↓
AWS / GCP / Azure / Kubernetes / Slurm / other supported infrastructure
    ↓
Observed execution evidence
    ↓
Kiste Record / GitOps history
```

Canonical rule:

```text
Kiste decides what is acceptable.
SkyPilot decides what is obtainable.
```

---

## 3. Why SkyPilot Is a Tool, Not a Provider

Kiste v0.9.10 removed Provider as a first-class model. SkyPilot fits that corrected model directly:

```text
Capability = what Kiste needs
Tool = SkyPilot
KisteUnit = SkyPilot adapter/integration package
Workspace = policy-bound context
ResolvedCapability = selected SkyPilot-backed implementation
```

SkyPilot may implement many Kiste capabilities at once.

---

## 4. Initial Capability Surface

The SkyPilot integration unit MAY provide:

```text
compute.select
compute.accelerator
compute.cpu
compute.memory
compute.cloud
compute.kubernetes
compute.slurm
compute.vm
compute.spot
compute.cost_estimate
compute.failover
compute.managed_job
runtime.execute_approved
runtime.status
runtime.logs
```

It MAY require:

```text
workspace.read
policy.validate
review.validate
secret.ref
key.ref
artifact.resolve
compatibility.resolve
record.write
```

Kiste MUST NOT imply that SkyPilot proves software compatibility across accelerator families.

Example:

```text
CUDA artifact  -> H100/A100/L4 candidates
ROCm artifact  -> MI300X candidates
XLA artifact   -> TPU candidates
```

Kiste determines which artifact/resource combinations are valid. SkyPilot receives only the acceptable candidate set.

---

## 5. Tool Contract

```yaml
apiVersion: kiste.dev/v0.9.12
kind: Tool

metadata:
  name: skypilot

spec:
  category: heterogeneous-compute

  interface:
    type: existing-sdk-api
    language: python
    sdk_compatible: true
    cli_compatible: true

  can_implement:
    capabilities:
      - compute.select
      - compute.accelerator
      - compute.cloud
      - compute.kubernetes
      - compute.slurm
      - compute.vm
      - compute.spot
      - compute.cost_estimate
      - compute.failover
      - compute.managed_job
      - runtime.execute_approved
      - runtime.status
```

---

## 6. KisteUnit Contract

```yaml
apiVersion: kiste.dev/v0.9.12
kind: KisteUnit

metadata:
  name: skypilot-integration-unit
  module: github.com/KisteBox/kiste-unit-skypilot

spec:
  category: existing-tool-integration

  integrates:
    tools:
      - skypilot

  provides:
    capabilities:
      - compute.select
      - compute.accelerator
      - compute.cloud
      - compute.kubernetes
      - compute.slurm
      - compute.vm
      - compute.spot
      - compute.cost_estimate
      - compute.failover
      - compute.managed_job
      - runtime.execute_approved
      - runtime.status

  requires:
    capabilities:
      - workspace.read
      - policy.validate
      - review.validate
      - secret.ref
      - key.ref
      - artifact.resolve
      - compatibility.resolve
      - record.write

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

## 7. ExecutionPlan Is the Boundary Object

Kiste SHOULD introduce or stabilize a backend-neutral `ExecutionPlan` object.

```yaml
apiVersion: kiste.dev/v0.9.12
kind: ExecutionPlan

metadata:
  id: job-8274

spec:
  artifact:
    ref: oci://example/model@sha256:abc
    compatibility_profile: cuda12

  resources:
    accelerators:
      acceptable:
        - H100:1
        - A100-80GB:1
        - L40S:1
    cpu: ">=8"
    memory: ">=64GB"

  placement:
    allowed_regions:
      - ap-southeast-1
      - asia-southeast1

  policy:
    internet: restricted
    max_cost_usd: 25

  execution:
    backend: skypilot
    managed: true

  command:
    run: python serve.py
```

Kiste compiles this object into SkyPilot's task/resource model.

The `ExecutionPlan` MUST remain backend-neutral enough that another backend can implement the same capability later.

---

## 8. Translation Rule

The SkyPilot adapter SHOULD map:

```text
Kiste ExecutionPlan
  artifact             -> SkyPilot workdir/image/file mounts as applicable
  command              -> Task.run
  setup                -> Task.setup
  acceptable resources -> Resources / resource alternatives
  region constraints   -> Resources region/infra constraints
  spot preference      -> use_spot
  labels               -> SkyPilot resource labels
  managed job          -> managed jobs API
```

Kiste-specific objects MUST NOT be flattened away:

```text
policy
review decision
execution contract
artifact evidence
compatibility evidence
Git commit
record identity
```

These remain Kiste-owned metadata referenced by the execution record.

---

## 9. Lifecycle Mapping

### read

Kiste MAY detect SkyPilot availability and version without mutating infrastructure.

### inspect

Kiste MAY inspect:

```text
available SkyPilot SDK/CLI
configured infrastructure backends
workspace policy
supported acceptable resource families
```

No cloud resource creation is allowed.

### plan

Kiste resolves:

```text
logical workload
  -> validated artifact
  -> acceptable resource candidates
  -> SkyPilot-backed ExecutionPlan
```

SkyPilot may be asked for dry-run/cost/feasibility information where supported, but mutation remains disabled.

### review

Kiste review gates the ExecutionPlan, including:

```text
policy
cost ceiling
regions
credentials/authority references
artifact identity
candidate resources
mutation boundary
```

### deploy

Only an approved plan may invoke SkyPilot mutation/execution.

### monitor

SkyPilot job/cluster status and logs are normalized into Kiste monitor signals and execution evidence.

---

## 10. GitOps Boundary

SkyPilot is not the GitOps source of truth.

Kiste retains Git-native desired state and decision history:

```text
Git commit / PR
    ↓
Kiste plan
    ↓
Kiste review
    ↓
approved ExecutionPlan
    ↓
SkyPilot
    ↓
execution
    ↓
Kiste Record
```

SkyPilot's operational state is observed runtime state. Kiste/Git stores consequential desired state and authorization history.

---

## 11. Credentials and Key Management

The integration MUST NOT turn Kiste into a long-lived cloud key store.

Preferred model:

```text
Kiste authority decision
    ↓
IAM / STS / workload identity / Vault / cloud-native identity
    ↓
short-lived or referenced credentials
    ↓
SkyPilot
```

Kiste stores references and authority metadata, not plaintext secret values. SkyPilot may consume the credentials/identities required to execute the approved plan.

---

## 12. Cost and OpenCost

SkyPilot cost information is primarily planning/placement input.

For Kubernetes execution, Kiste MAY integrate OpenCost as a separate observation tool:

```text
Kiste plan
   ↓
SkyPilot estimated placement cost
   ↓
Kubernetes execution
   ↓
OpenCost observed allocation cost
   ↓
Kiste Record
```

Kiste SHOULD distinguish:

```text
estimated_cost
actual_cost
cost_source
cost_prediction_error
```

This allows Kiste to learn job-level economics without replacing SkyPilot or OpenCost.

---

## 13. Passthrough

Kiste SHOULD permit bounded SkyPilot-specific passthrough configuration.

```yaml
execution:
  backend: skypilot
  passthrough:
    resources:
      use_spot: true
```

Passthrough MUST:

```text
be namespaced to the backend
remain policy-visible
remain reviewable
never bypass Kiste approval or secret boundaries
```

Portable Kiste semantics remain preferred, but Kiste MUST NOT pretend every backend-specific feature can be losslessly abstracted.

---

## 14. Non-Goals

Kiste MUST NOT reimplement SkyPilot functionality solely for abstraction purity.

Non-goals include:

```text
cloud GPU catalog maintenance
cloud instance catalog maintenance
region/zone availability probing
multi-cloud provisioning logic
spot recovery
cloud failover
Slurm submission logic already handled by SkyPilot
Kubernetes placement logic already handled by SkyPilot
SkyPilot cost optimizer duplication
```

If a required feature already exists robustly in SkyPilot, Kiste should integrate it before considering a native implementation.

---

## 15. Fallback and Replaceability

SkyPilot is the preferred initial heterogeneous compute backend, not a permanent hard dependency of Kiste semantics.

```text
ExecutionPlan
   ├── SkyPilot backend
   ├── native Kubernetes backend
   ├── future Slurm backend
   └── future specialized backend
```

Workspace capability resolution decides which backend is allowed.

This preserves Kiste's higher-level architecture if SkyPilot changes, is unavailable, or a superior backend appears.

---

## 16. Acceptance Criteria

The integration is accepted only if:

```text
1. SkyPilot is modeled as a Tool integrated by a KisteUnit.
2. No new Provider abstraction is introduced.
3. Kiste core does not contain SkyPilot-specific scheduling logic.
4. Kiste determines compatibility/acceptable candidate resources before delegation.
5. SkyPilot owns placement, provisioning, failover, and managed execution.
6. Mutation requires an approved Kiste plan.
7. Secret values are never required in Kiste manifests.
8. SkyPilot outputs can be normalized into Kiste records/evidence.
9. Backend-specific passthrough is explicit and policy-visible.
10. The ExecutionPlan boundary is usable by a non-SkyPilot backend.
11. Git remains Kiste's desired-state/decision-history boundary.
12. Existing SkyPilot functionality is reused instead of reimplemented without a documented reason.
```

---

## 17. Final Rule

```text
Kiste does not compete with SkyPilot on compute orchestration.

Kiste resolves intent, compatibility, policy, trust, and authorization.
SkyPilot executes the approved heterogeneous-compute plan.

Kiste decides what is acceptable.
SkyPilot decides what is obtainable.
```
