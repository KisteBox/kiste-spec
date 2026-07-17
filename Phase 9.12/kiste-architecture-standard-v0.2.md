# Kiste Architecture Standard v0.2

## 1. Core principle

The **Kiste Unit is the nucleus of Kiste**.

Applications, infrastructure, databases, policies, optimizers, identity systems, key managers and the Kiste control plane are all represented as Units.

A Unit is:

> An independently identifiable, permission-bounded, capability-declaring, lifecycle-managed and auditable part of a system.

A Unit may be physically implemented as a directory, repository, container, process, service, cluster or collection of these. Physical packaging does not define the Unit boundary.

---

## 2. Universal object model

Every Unit has the same fundamental structure:

```text
Kiste Unit
├── Identity
├── Inputs
├── Outputs
├── Capabilities
├── Permissions
├── Resources
├── Relationships
├── Six-step lifecycle
├── State
└── Evidence
```

The same model applies to:

```text
Application Unit
Database Unit
Infrastructure Unit
Policy Unit
Control Plane Unit
Identity Unit
Key Manager Unit
Optimizer Unit
Runtime Unit
Hardware Unit
Workspace Unit
```

Special Kiste components are not exceptions to the Unit model. They are **system-class Units** with stronger permissions and trust requirements.

---

## 3. Standard Unit classes

Use `spec.class` to define the operational class.

```yaml
spec:
  class: workload
```

Standard classes:

| Class | Purpose |
|---|---|
| `workload` | Application, service, worker or user workload |
| `resource` | Database, bucket, network, runtime or infrastructure |
| `policy` | Policy, compliance or governance logic |
| `system` | Kiste control-plane or trusted platform component |
| `composite` | Unit composed of other Units |
| `external` | System outside Kiste control |

Use `spec.role` for the Unit’s main role.

```yaml
spec:
  class: system
  role: control-plane
```

Initial standard roles:

```text
application
service
library
database
infrastructure
runtime
policy-engine
identity
key-manager
optimizer
scheduler
state-store
evidence-store
control-plane
workspace
generic
```

`class` controls trust and execution rules.

`role` describes what the Unit does.

---

## 4. The six-step lifecycle

Every Unit supports the same lifecycle:

```text
1. Read
2. Inspect
3. Plan
4. Recommend
5. Apply
6. Recheck
```

Canonical names:

```yaml
lifecycle:
  workflow: kiste.standard/v1

  steps:
    read: {}
    inspect: {}
    plan: {}
    recommend: {}
    apply: {}
    recheck: {}
```

### 4.1 Read

Purpose:

```text
Resolve identity
Resolve source revisions
Resolve dependencies
Resolve capability providers
Load current state
Load permissions
Materialize declared inputs
```

Outputs:

```text
Resolved Unit graph
Resolved revisions
Resolved capabilities
Observed state
Permission context
```

### 4.2 Inspect

Purpose:

```text
Analyze source
Analyze infrastructure
Analyze dependencies
Analyze security
Analyze permissions
Analyze runtime and hardware
Detect drift
Detect compatibility problems
```

Outputs:

```text
Findings
Risks
Capability inventory
Dependency graph
Permission findings
Drift report
```

### 4.3 Plan

Purpose:

```text
Calculate required changes
Order operations
Estimate impact
Calculate required permissions
Create rollback strategy
```

Outputs:

```text
Execution plan
Permission requirements
Resource changes
Dependency order
Rollback plan
```

### 4.4 Recommend

Purpose:

```text
Compare valid plans
Apply optimizer preferences
Explain trade-offs
Collect user or policy changes
Request approvals
```

Outputs:

```text
Recommended plan
Alternative plans
User modifications
Approvals
Rejected actions
```

The recommendation step may be fully automatic, human-reviewed or policy-controlled.

### 4.5 Apply

Purpose:

```text
Execute the approved plan
Change resources
Build or deploy artifacts
Update declared state
Perform rollback when necessary
```

Outputs:

```text
Action results
Changed resources
Deployment results
Rollback results
Execution evidence
```

### 4.6 Recheck

Purpose:

```text
Read the resulting system again
Verify desired state
Verify security
Verify health
Verify performance
Detect remaining drift
```

Outputs:

```text
Verified state
Remaining drift
Health evidence
Security evidence
Performance evidence
Final result
```

---

## 5. Lifecycle invariants

The following rules are mandatory:

1. `inspect` must use the state resolved by `read`.
2. `plan` must be based on inspection findings.
3. `apply` must use an approved plan.
4. `apply` must not request undeclared permissions silently.
5. `recheck` must inspect the actual applied result.
6. Every step must produce evidence.
7. A failed step must not erase evidence from earlier steps.
8. Production changes must support explicit approval policies.
9. A Unit may customize a step, but may not change the meaning of the standard lifecycle.
10. A system Unit is subject to the same lifecycle as a workload Unit.

---

## 6. Unit input and output model

Units interact through declared interfaces.

```text
Unit A output
      ↓
Binding
      ↓
Unit B input
```

A Unit must not depend on undeclared files, environment variables, network endpoints or secrets.

### Standard input definition

```yaml
interfaces:
  inputs:
    source:
      type: source.repository
      required: true

    database:
      type: database.connection
      required: true
      sensitive: true

    environment:
      type: string
      required: true
```

### Standard output definition

```yaml
interfaces:
  outputs:
    image:
      type: container.image
      immutable: true

    endpoint:
      type: service.endpoint

    evidence:
      type: evidence.bundle
      immutable: true
```

### Standard interface types

```text
string
number
integer
boolean
object
array

file
directory
stream
event

source.repository
artifact.package
container.image

service.endpoint
database.connection

secret.reference
key.reference
identity.reference

unit.reference
resource.reference
capability.reference

plan
evidence.bundle
```

Sensitive outputs expose references, not secret values.

Correct:

```yaml
type: secret.reference
```

Incorrect:

```yaml
value: plaintext-password
```

---

## 7. Capability model

Capabilities describe behavior.

A Unit declares:

```yaml
capabilities:
  provides: []
  requires: []
  optional: []
  conflicts: []
```

Example:

```yaml
capabilities:
  provides:
    - id: payment.api
      version: 1.0.0

  requires:
    - id: database.connection
      version: "^1.0"

    - id: identity.token.validate
      version: "^1.0"

    - id: secret.read
      version: "^1.0"
```

Capability IDs must describe behavior rather than vendors.

Preferred:

```text
source.clone
artifact.publish
database.connection
secret.read
identity.authenticate
container.build
deployment.apply
evidence.store
```

Avoid:

```text
gitlab.clone
aws.s3.upload
vault.get-secret
```

Vendor-specific capabilities are allowed only when the behavior cannot be expressed portably.

---

## 8. Capability binding

A requirement is satisfied by a provider Unit.

```yaml
bindings:
  - name: database
    requires:
      capability: database.connection

    provider:
      unitRef: payments/payment-database

    input:
      fromOutput: connection
      toInput: database
```

The resulting graph is:

```text
payment-api
    requires database.connection
        ↓
payment-database
    provides database.connection
```

Kiste must verify:

```text
Capability identity
Version compatibility
Permission compatibility
Input/output compatibility
Trust level
Environment compatibility
Platform compatibility
```

---

## 9. Permission model

Kiste permissions are capability-based and resource-scoped.

Every authorization decision contains:

```text
Subject
Action
Resource
Context
Effect
```

Example:

```text
Subject: payments/payment-api
Action: secret.read
Resource: secret/payment-database-password
Context: production
Effect: allow
```

### Standard permission actions

```text
discover
read
consume
produce
invoke
create
update
delete
apply
approve
manage
delegate
sign
verify
encrypt
decrypt
assume
impersonate
```

### Permission layers

Kiste uses four permission layers:

| Layer | Controls |
|---|---|
| Interface permission | Which inputs and outputs may be connected |
| Capability permission | Which behavior a Unit may invoke |
| Resource permission | Which concrete resources may be accessed |
| Lifecycle permission | Which lifecycle steps a Unit may execute or approve |

---

## 10. Permission requests

A Unit declares the permissions it needs.

```yaml
permissions:
  requests:
    - action: source.read
      resourceRef: application-source

    - action: secret.read
      resourceRef: database-password
      conditions:
        environment: production

    - action: deployment.apply
      resourceRef: payment-runtime
      conditions:
        approval: required
```

A request is not automatically granted.

The identity or policy Unit evaluates it.

---

## 11. Permission grants

A grant connects a subject Unit to an allowed action and resource.

```yaml
permissions:
  grants:
    - subject:
        unitRef: payments/payment-api

      actions:
        - secret.read

      resources:
        - secretRef: payment-database-password

      conditions:
        environment: production
        lifecycleStep: apply

      effect: allow
```

Rules:

1. Default effect is `deny`.
2. Explicit deny overrides allow.
3. Grants must be resource-scoped.
4. Grants may be step-scoped.
5. Grants may be environment-scoped.
6. Grants may expire.
7. Delegation must be explicitly allowed.
8. Unit ownership does not automatically grant administrative permission.
9. System Units must not have unrestricted access by default.
10. All permission decisions must produce evidence.

---

## 12. Lifecycle permissions

Each lifecycle step has different default permission limits.

| Step | Default permission level |
|---|---|
| `read` | Read declared metadata and inputs |
| `inspect` | Read and analyze; no mutation |
| `plan` | Simulate changes; no mutation |
| `recommend` | Propose and request approval |
| `apply` | Use explicitly approved mutation permissions |
| `recheck` | Read resulting state and execute verification probes |

An inspection plugin must not silently perform deployment actions.

A planning plugin must not silently update infrastructure.

An apply operation must not use permissions that were absent from the approved plan.

---

## 13. The control plane as a Unit

The Kiste control plane is a system Unit.

```yaml
apiVersion: kiste.dev/v1alpha1
kind: Unit

metadata:
  name: control-plane
  namespace: kiste-system

spec:
  class: system
  role: control-plane
```

It may provide:

```yaml
capabilities:
  provides:
    - id: lifecycle.coordinate
    - id: capability.resolve
    - id: execution.schedule
    - id: plan.evaluate
    - id: evidence.coordinate
    - id: state.reconcile
```

It should not automatically own:

```text
All encryption keys
All user identities
All secrets
All resources
All approval authority
```

The control plane coordinates authority. It should not necessarily contain every authority.

---

## 14. System Units

Recommended logical system Units:

```text
kiste-system/control-plane
kiste-system/identity
kiste-system/key-manager
kiste-system/policy-engine
kiste-system/state-store
kiste-system/evidence-store
kiste-system/optimizer
kiste-system/execution-runtime
```

### Control Plane Unit

Responsibilities:

```text
Coordinate lifecycle
Resolve Unit graphs
Schedule executions
Coordinate reconciliation
Track executions
```

### Identity Unit

Responsibilities:

```text
Authenticate users and Units
Issue Unit identities
Validate tokens
Manage roles and trust relationships
```

### Key Manager Unit

Responsibilities:

```text
Generate keys
Store key references
Sign and verify
Encrypt and decrypt
Rotate keys
Control key usage
```

### Policy Engine Unit

Responsibilities:

```text
Evaluate permission requests
Evaluate organizational policy
Evaluate approval requirements
Return allow or deny decisions
```

### State Store Unit

Responsibilities:

```text
Store desired and observed state
Store execution metadata
Support locking and reconciliation
```

### Evidence Store Unit

Responsibilities:

```text
Store immutable lifecycle evidence
Store reports and attestations
Verify evidence digests
```

### Optimizer Unit

Responsibilities:

```text
Compare implementation choices
Evaluate cost, performance, security and privacy
Select compatible providers
Recommend plans
```

### Execution Runtime Unit

Responsibilities:

```text
Run lifecycle actions
Provide isolation
Materialize inputs
Collect outputs
Enforce execution permissions
```

---

## 15. Logical separation and physical merging

Kiste must distinguish:

```text
Logical Unit boundary
```

from:

```text
Physical deployment boundary
```

Several logical Units may run in one binary, process, container or virtual machine.

For the MVP, this is acceptable:

```text
kiste-core process
├── control-plane Unit
├── optimizer Unit
├── policy-engine Unit
├── local identity Unit
├── local key-manager Unit
├── state-store Unit
└── execution-runtime Unit
```

They remain separate logical Units with separate:

```text
Capabilities
Permissions
Inputs and outputs
State
Evidence
Trust responsibilities
```

Later, the same logical Units may be split into separate services without changing the Unit standard.

---

## 16. Recommended merge strategy

### MVP mode

Use one deployable Kiste system bundle:

```text
Kiste Core Bundle
├── Control plane
├── Optimizer
├── Policy engine
├── Local identity
├── Local key manager
├── SQLite or file state
└── Local execution runtime
```

This reduces deployment complexity.

### Production mode

Split high-trust components:

```text
Control plane
Identity provider
Key manager
Policy engine
State store
Evidence store
Execution workers
```

### Components that may be merged safely at first

```text
Control plane + scheduler
Control plane + graph resolver
Control plane + lifecycle coordinator
Optimizer + recommendation engine
State store + execution metadata store
Local development identity + local policy engine
```

### Components that should remain logically separate

```text
Identity
Key management
Policy decisions
Execution runtime
Approval authority
Evidence storage
```

Even when physically bundled, they should expose separate internal capability interfaces.

---

## 17. Why IAM and key management should not become ordinary control-plane functions

Combining everything without logical boundaries creates one unlimited authority:

```text
Control Plane
├── identifies everyone
├── grants every permission
├── owns every encryption key
├── executes every change
└── verifies its own evidence
```

That produces a dangerous trust loop.

A safer design is:

```text
Control Plane requests
Policy Engine decides
Identity Unit identifies
Key Manager performs cryptographic operation
Execution Runtime executes
Evidence Store records
Recheck verifies
```

No single component should silently:

```text
Request
Approve
Execute
And verify
```

the same sensitive operation.

---

## 18. Bootstrap mode

A fresh Kiste installation needs an initial trusted Unit.

```yaml
spec:
  class: system
  role: control-plane

  trust:
    mode: bootstrap
```

Bootstrap mode may temporarily provide:

```text
Local identity
Local key management
Local policy evaluation
Local state
Local execution
```

Restrictions:

```text
Bootstrap mode must be explicit
Bootstrap credentials must be replaceable
Production use should produce warnings
Migration to external identity and key management must be supported
Bootstrap authority must be auditable
```

After migration:

```yaml
trust:
  mode: federated

  identityUnitRef: kiste-system/company-identity
  keyManagerUnitRef: kiste-system/company-key-manager
  policyUnitRef: kiste-system/company-policy
```

---

## 19. Standard Unit YAML

```yaml
apiVersion: kiste.dev/v1alpha1
kind: Unit

metadata:
  name: payment-api
  namespace: payments
  version: 1.0.0

spec:
  class: workload
  role: application

  interfaces:
    inputs:
      source:
        type: source.repository
        required: true

      database:
        type: database.connection
        required: true
        sensitive: true

      signingKey:
        type: key.reference
        required: true
        sensitive: true

    outputs:
      image:
        type: container.image
        immutable: true

      endpoint:
        type: service.endpoint

      evidence:
        type: evidence.bundle
        immutable: true

  capabilities:
    provides:
      - id: payment.api
        version: 1.0.0

    requires:
      - id: source.read
        version: "^1.0"

      - id: database.connection
        version: "^1.0"

      - id: container.build
        version: "^1.0"

      - id: deployment.apply
        version: "^1.0"

      - id: key.sign
        version: "^1.0"

  permissions:
    requests:
      - action: source.read
        resourceRef: application-source

      - action: database.connect
        resourceRef: payment-database

      - action: key.sign
        resourceRef: release-signing-key
        conditions:
          lifecycleStep: apply

      - action: deployment.apply
        resourceRef: payment-production
        conditions:
          environment: production
          approval: required

  resources:
    application-source:
      type: git.repository

      location:
        providerRef: company-git
        repository: payments/payment-api

      revision:
        branch: main

    payment-production:
      type: compute.runtime

      location:
        providerRef: production-runtime
        target: payment-cluster

  bindings:
    - name: database
      requires:
        capability: database.connection

      provider:
        unitRef: payments/payment-database

      interface:
        fromOutput: connection
        toInput: database

    - name: signing
      requires:
        capability: key.sign

      provider:
        unitRef: kiste-system/key-manager

      interface:
        fromOutput: signing-key-reference
        toInput: signingKey

  lifecycle:
    workflow: kiste.standard/v1

    steps:
      read:
        permissions:
          mode: read-only

      inspect:
        checks:
          - source
          - dependencies
          - security
          - permissions
          - infrastructure
          - runtime
          - performance
          - hardware

      plan:
        requirePermissionDiff: true
        requireRollbackPlan: true

      recommend:
        optimizerUnitRef: kiste-system/optimizer
        approvalPolicyRef: production-policy

      apply:
        approval:
          requiredFor:
            - production

      recheck:
        verify:
          - desired-state
          - health
          - security
          - permissions
          - performance

  policies:
    permissions:
      defaultEffect: deny
      undeclaredAccess: deny

    secrets:
      inlineValues: deny

    execution:
      privileged: deny

    audit:
      recordAllSteps: true
      evidenceRequired: true
```

---

## 20. Control Plane Unit example

```yaml
apiVersion: kiste.dev/v1alpha1
kind: Unit

metadata:
  name: control-plane
  namespace: kiste-system
  version: 0.1.0

spec:
  class: system
  role: control-plane

  trust:
    level: system
    mode: bootstrap

  interfaces:
    inputs:
      unitSpecifications:
        type: stream

      lifecycleRequests:
        type: event

      policyDecisions:
        type: stream

      identityAssertions:
        type: identity.reference

    outputs:
      executionPlans:
        type: plan
        immutable: true

      lifecycleEvents:
        type: event

      executionEvidence:
        type: evidence.bundle
        immutable: true

  capabilities:
    provides:
      - id: lifecycle.coordinate
      - id: capability.resolve
      - id: unit.graph.resolve
      - id: execution.schedule
      - id: state.reconcile

    requires:
      - id: identity.authenticate
      - id: policy.evaluate
      - id: state.read
      - id: state.write
      - id: evidence.store
      - id: execution.run

  bindings:
    - name: identity
      provider:
        unitRef: kiste-system/identity

    - name: policy
      provider:
        unitRef: kiste-system/policy-engine

    - name: keys
      provider:
        unitRef: kiste-system/key-manager

    - name: state
      provider:
        unitRef: kiste-system/state-store

    - name: evidence
      provider:
        unitRef: kiste-system/evidence-store

    - name: execution
      provider:
        unitRef: kiste-system/execution-runtime

  permissions:
    requests:
      - action: unit.discover
        resourceSelector:
          namespace: "*"

      - action: lifecycle.coordinate
        resourceSelector:
          class:
            - workload
            - resource
            - composite

      - action: state.write
        resourceRef: kiste-state

      - action: evidence.produce
        resourceRef: kiste-evidence

  policies:
    permissions:
      maySelfApprove: false
      mayReadSecretValues: false
      mayExportKeys: false
      mayModifyEvidence: false
```

---

## 21. Project structure

Recommended repository layout:

```text
kiste/
├── core/
│   ├── unit/
│   ├── lifecycle/
│   ├── capabilities/
│   ├── permissions/
│   ├── interfaces/
│   ├── graph/
│   ├── state/
│   └── evidence/
│
├── system-units/
│   ├── control-plane/
│   ├── identity/
│   ├── key-manager/
│   ├── policy-engine/
│   ├── state-store/
│   ├── evidence-store/
│   ├── optimizer/
│   └── execution-runtime/
│
├── providers/
│   ├── git/
│   ├── object-store/
│   ├── runtime/
│   ├── identity/
│   ├── secrets/
│   └── infrastructure/
│
├── schemas/
│   ├── unit.schema.json
│   ├── capability.schema.json
│   ├── permission.schema.json
│   └── evidence.schema.json
│
├── workflows/
│   └── standard-v1/
│
├── cli/
├── sdk/
└── tests/
```

---

## 22. Core implementation order

The first implementation should focus on:

```text
1. Unit parser and validation
2. Six-step lifecycle engine
3. Input and output interfaces
4. Capability declaration and matching
5. Permission requests and grants
6. Unit dependency graph
7. Execution evidence
8. Local control-plane system Unit
9. Local policy and identity modules
10. Provider adapters
```

Do not initially build:

```text
Distributed control plane
Cross-cluster runtime sharing
Advanced container cache sharing
Complex federation
Multi-region consensus
Custom cryptographic service
Fully autonomous optimizer
```

Those can be added without changing the Unit contract.

---

## 23. Final architectural rules

### Unit rule

Everything that owns a lifecycle boundary is a Unit.

### Lifecycle rule

Every Unit follows:

```text
Read → Inspect → Plan → Recommend → Apply → Recheck
```

### Interface rule

All Unit interactions occur through declared inputs, outputs, capabilities and bindings.

### Permission rule

No capability invocation or resource access is implicitly allowed.

### System Unit rule

The control plane, identity system, key manager, policy engine and optimizer are Units with system-level roles.

### Packaging rule

Logical Units may be deployed together, but their contracts and permissions remain separate.

### Trust rule

The component that requests an action should not automatically approve, execute and verify the same sensitive action.

### Portability rule

Units declare required behavior. Providers implement that behavior.

### Audit rule

Every lifecycle step, permission decision and applied change produces evidence.

### Evolution rule

The first Kiste implementation may be one binary, while the architecture remains ready to split into independently deployed Units later.
