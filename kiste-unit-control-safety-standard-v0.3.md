# Kiste Unit and Control Safety Standard v0.3

## 1. Normative definition of a Kiste Unit

A **Kiste Unit** is the smallest independently identifiable Kiste object that owns:

- a declared purpose;
- inputs and outputs;
- provided and required capabilities;
- permissions;
- sources and resources;
- relationships with other Units;
- the six-step lifecycle;
- desired and observed state;
- version history;
- execution evidence.

A Unit is defined by its **operational boundary**, not by its physical size.

A Unit may correspond to:

```text
One file
One directory
Part of a monorepo
One repository
Several repositories
A service and its infrastructure
A database and its migrations
A machine-learning model and runtime
A control-plane service
A complete workspace
```

A Unit must be independently:

```text
Identifiable
Versionable
Inspectable
Plannable
Permission-bounded
Executable
Recheckable
Auditable
Composable
```

### Canonical definition

> A Kiste Unit is a versioned, capability-declaring, permission-bounded and auditable lifecycle object that transforms declared inputs and resources into declared outputs and observed state through the standard Kiste six-step cycle.

---

## 2. Canonical Unit identity

```yaml
apiVersion: kiste.dev/v1alpha1
kind: Unit

metadata:
  name: payment-api
  namespace: payments
  version: 1.2.0
```

The canonical Unit identity is:

```text
namespace/name
```

Example:

```text
payments/payment-api
```

A version-specific identity is:

```text
namespace/name@version
```

Example:

```text
payments/payment-api@1.2.0
```

The Unit contract version is separate from:

```text
Git commit
Git tag
Application version
Container image tag
Database schema version
Deployment revision
```

Kiste links these versions together through evidence.

---

## 3. Required Unit model

Every Unit has the following conceptual structure:

```text
Kiste Unit
├── Identity
├── Classification
├── Sources
├── Interfaces
│   ├── Inputs
│   └── Outputs
├── Capabilities
│   ├── Provides
│   ├── Requires
│   ├── Optional
│   └── Conflicts
├── Permissions
│   ├── Requests
│   └── Grants
├── Resources
├── Relationships
├── Lifecycle
│   ├── Read
│   ├── Inspect
│   ├── Plan
│   ├── Recommend
│   ├── Apply
│   └── Recheck
├── Desired State
├── Observed State
└── Evidence
```

Canonical YAML structure:

```yaml
apiVersion: kiste.dev/v1alpha1
kind: Unit

metadata: {}

spec:
  class: workload
  role: generic

  sources: {}
  interfaces:
    inputs: {}
    outputs: {}

  capabilities:
    provides: []
    requires: []
    optional: []
    conflicts: []

  permissions:
    requests: []
    grants: []

  resources: {}
  relationships: []
  lifecycle: {}
  policies: {}

status:
  phase: Unknown
  resolved: {}
  conditions: []
  evidence: {}
```

---

## 4. Git as the primary auditable history of a Unit

Git is a first-class part of the Kiste Unit model.

Git records:

```text
Who proposed a Unit change
What changed
When it changed
Why it changed
Which revision was inspected
Which revision was planned
Which revision was approved
Which revision was applied
Which revision was rechecked
```

A Unit should normally have at least one Git-backed source:

```yaml
spec:
  sources:
    primary:
      type: git.repository
      role: definition

      location:
        providerRef: company-git
        repository: payments/payment-api
        path: .

      revision:
        branch: main
```

During `read`, Kiste resolves the mutable branch to an immutable commit:

```yaml
status:
  resolved:
    sources:
      primary:
        requested:
          branch: main

        commit: 842ef60726adbe42c11a6147e705ddb8b3290057
```

Every later lifecycle step must reference that exact commit unless a new lifecycle execution begins.

---

## 5. Git audit invariants

A Kiste lifecycle execution must record:

```text
Repository identity
Requested branch, tag or reference
Resolved commit SHA
Tree digest
Subdirectory path
Submodule revisions
Configuration digest
Policy digest
Unit specification digest
```

Example evidence:

```yaml
status:
  evidence:
    source:
      repository: payments/payment-api
      requestedRef: refs/heads/main
      commit: 842ef60726adbe42c11a6147e705ddb8b3290057
      treeDigest: sha256:7fe3...
      unitSpecDigest: sha256:14ab...
```

### Required Git controls for protected environments

```yaml
spec:
  policies:
    git:
      protectedReferences:
        required: true

      forcePush:
        effect: deny

      signedCommits:
        requiredFor:
          - production

      signedTags:
        requiredFor:
          - releases

      minimumApprovals:
        production: 2

      immutableApplyRevision:
        required: true
```

For production, Kiste should reject an apply operation when:

```text
The resolved commit no longer matches the approved plan
The branch was force-pushed after approval
The Unit specification changed
A required signature is missing
Required approvals are missing
The policy digest changed
The source tree digest changed
```

---

## 6. Git is necessary but not sufficient

Git history can be rewritten by sufficiently privileged users. Therefore, Git must not be the only evidence store.

Kiste auditability uses two linked systems:

```text
Git
  ├── desired state
  ├── source history
  ├── review history
  └── declared configuration

Immutable evidence store
  ├── resolved commit
  ├── inspection report
  ├── approved plan
  ├── permission decisions
  ├── apply result
  └── recheck result
```

Each evidence object should be content-addressed:

```yaml
evidence:
  plan:
    digest: sha256:09cb...

  approval:
    digest: sha256:c913...

  apply:
    digest: sha256:f773...

  recheck:
    digest: sha256:a110...
```

The final execution evidence should bind Git and runtime reality together:

```text
Unit specification digest
        +
Git commit SHA
        +
Plan digest
        +
Approval digest
        +
Applied resource revisions
        +
Recheck digest
```

This creates an auditable chain from declared intent to observed result.

---

## 7. Git graph as part of the Unit graph

Each Unit owns or references a Git graph:

```text
Commit history
Branch relationships
Tags
Merge requests or pull requests
Approvals
Release references
Deployment references
```

The Git graph connects to the other Unit graphs:

```text
Git graph
    │
    ├── defines Unit specification
    ├── changes capability requirements
    ├── changes permissions
    ├── changes resource declarations
    └── triggers lifecycle execution

Capability graph
Permission graph
Resource graph
Execution graph
Evidence graph
```

Kiste should be able to answer:

```text
Which commit introduced this capability?
Which commit requested this permission?
Which plan applied this commit?
Who approved the plan?
Which runtime revision resulted from it?
Did recheck confirm the expected state?
```

---

## 8. The Kiste Control Unit

The Kiste control plane is represented as a system-class Unit.

```yaml
apiVersion: kiste.dev/v1alpha1
kind: Unit

metadata:
  name: control
  namespace: kiste-system
  version: 0.3.0

spec:
  class: system
  role: control-plane
```

The Control Unit coordinates:

```text
Unit discovery
Graph resolution
Capability matching
Lifecycle coordination
Permission requests
Planning
Scheduling
State reconciliation
Evidence collection
Failure handling
```

The Control Unit does not automatically own:

```text
All identities
All permissions
All keys
All secrets
All approvals
All execution environments
```

Those responsibilities may be provided by other system Units.

---

## 9. Fail-safe principle

The Kiste Control Unit must be designed to **fail closed**.

> When the Control Unit cannot prove that an action is authorized, current, safe and recoverable, it must not perform the action.

The default failure decision is:

```text
Unknown → deny mutation
Uncertain → stop
State conflict → stop
Permission failure → deny
Evidence failure → stop
Recheck failure → contain or rollback
```

Read-only inspection may continue in degraded mode where safe.

Mutation must stop when certainty is lost.

---

## 10. Control Unit safety states

The Control Unit has explicit operating states:

```text
Starting
Healthy
Degraded
SafeMode
Applying
RollingBack
Suspended
Failed
Recovering
```

Example:

```yaml
status:
  phase: SafeMode

  conditions:
    - type: StateStoreAvailable
      status: "False"

    - type: MutationAllowed
      status: "False"

    - type: ReadOnlyInspectionAllowed
      status: "True"
```

### State meanings

| State | Behavior |
|---|---|
| `Healthy` | All normal lifecycle operations allowed |
| `Degraded` | Limited operations; no high-risk changes |
| `SafeMode` | Read and inspect only |
| `Applying` | Executing an approved immutable plan |
| `RollingBack` | Reversing an incomplete or failed apply |
| `Suspended` | Operator or policy has stopped execution |
| `Failed` | Control Unit cannot safely continue |
| `Recovering` | Validating state before normal operation resumes |

---

## 11. Mandatory fail-safe controls

### 11.1 Fail-closed authorization

No permission decision means no action.

```yaml
spec:
  safety:
    authorization:
      defaultEffect: deny
      unavailablePolicyEngine: deny-mutations
      expiredIdentity: deny
      unverifiableIdentity: deny
```

The Control Unit must never interpret:

```text
Policy service unavailable
Identity service unavailable
Malformed grant
Expired token
Unknown capability
```

as permission to continue.

### 11.2 Immutable apply plan

Before `apply`, Kiste freezes:

```text
Unit commit
Unit specification digest
Resolved capabilities
Provider versions
Permission set
Plan
Policy digest
Approval records
```

Example:

```yaml
status:
  activeExecution:
    unitCommit: 842ef607...
    unitSpecDigest: sha256:14ab...
    planDigest: sha256:09cb...
    policyDigest: sha256:73de...
    permissionDigest: sha256:8821...
```

If any value changes before or during apply, Kiste pauses or aborts.

### 11.3 Two-phase apply

Sensitive changes use two phases:

```text
Phase 1: Prepare
├── validate providers
├── validate permissions
├── lock target state
├── verify rollback path
├── verify approvals
└── stage changes

Phase 2: Commit
├── execute staged changes
├── record results
├── release lock
└── begin recheck
```

The Control Unit must not enter the commit phase unless preparation succeeds completely.

### 11.4 Execution leases

Mutation authority should be time-limited.

```yaml
spec:
  safety:
    executionLease:
      required: true
      duration: 15m
      renewable: true
```

If the lease expires:

```text
Do not start new actions
Stop at the nearest safe boundary
Do not assume authorization remains valid
Enter SafeMode or RollingBack
```

This prevents an abandoned or partitioned controller from continuing indefinitely.

### 11.5 Single-writer or coordinated locking

Kiste must prevent two controllers from applying conflicting plans to the same Unit or resource.

```yaml
spec:
  safety:
    locking:
      mode: lease
      scope:
        - unit
        - resource
      conflictAction: stop
```

A stale or conflicting lock must not be silently ignored.

### 11.6 Circuit breaker

Repeated provider or execution failures trigger automatic suspension.

```yaml
spec:
  safety:
    circuitBreaker:
      enabled: true
      failureThreshold: 3
      resetMode: manual-or-health-check
```

When open:

```text
No further mutation attempts
Read-only health checks allowed
Operator notification produced
Evidence preserved
```

### 11.7 Rollback and containment

Every mutation plan must declare one of:

```text
Rollback supported
Compensating action supported
Containment only
Irreversible
```

Example:

```yaml
lifecycle:
  steps:
    plan:
      requireRecoveryStrategy: true

    apply:
      onFailure:
        action: rollback
```

For irreversible operations:

```yaml
permissions:
  requests:
    - action: resource.delete
      resourceRef: production-database
      conditions:
        irreversible: true
        approvalCount: 2
        breakGlassNotAllowed: true
```

Kiste must clearly identify irreversible changes before approval.

### 11.8 Recheck failure handling

A successful provider response does not prove system success.

If `recheck` fails:

```text
Mark execution unverified
Prevent promotion
Preserve all evidence
Attempt rollback when safe
Otherwise contain the change
Notify responsible identity
Require explicit recovery
```

Example:

```yaml
lifecycle:
  steps:
    recheck:
      failurePolicy:
        markExecution: unverified
        blockPromotion: true
        rollbackIfSafe: true
        otherwise: contain
```

### 11.9 No self-approval

The Control Unit must not approve its own sensitive plan.

```yaml
spec:
  policies:
    separationOfDuties:
      controlMayRequest: true
      controlMayPlan: true
      controlMayExecute: true
      controlMaySelfApprove: false
```

For high-risk operations, the following identities should be distinct:

```text
Requester
Planner or recommender
Approver
Executor
Verifier
```

They may be automated Units, but their authority must remain logically separate.

### 11.10 Independent evidence recording

The Control Unit must not be able to silently alter its own past evidence.

```yaml
capabilities:
  requires:
    - id: evidence.append
    - id: evidence.verify

  conflicts:
    - id: evidence.rewrite
```

Evidence storage should support append-only or immutable records.

---

## 12. Safe degraded operation

When a dependency fails, the Control Unit may continue only operations proven safe.

| Missing dependency | Allowed behavior |
|---|---|
| Identity Unit | Public metadata read only |
| Policy Unit | Inspect only; deny mutation |
| Key Manager | Operations not requiring signing or decryption |
| State Store | Local diagnostic inspection only |
| Evidence Store | Stop mutation |
| Optimizer | Use previously approved plan or stop recommendation |
| Execution Runtime | Plan only |
| Git provider | Use already resolved immutable content only when policy permits |

A missing evidence store must block mutation because Kiste could not prove what occurred.

---

## 13. Manual emergency control

Kiste should provide explicit emergency operations:

```text
Suspend all mutations
Suspend one Unit
Revoke one execution lease
Open circuit breaker
Enter SafeMode
Trigger verified rollback
Resume after recovery validation
```

Emergency access must be:

```text
Explicit
Time-limited
Strongly authenticated
Narrowly scoped
Fully audited
```

Example:

```yaml
spec:
  safety:
    breakGlass:
      enabled: true
      maximumDuration: 30m
      requireReason: true
      requireStrongAuthentication: true
      recordAllActions: true
      allowEvidenceDeletion: false
      allowKeyExport: false
```

Break-glass access must not allow deletion of audit evidence.

---

## 14. Control Unit YAML

```yaml
apiVersion: kiste.dev/v1alpha1
kind: Unit

metadata:
  name: control
  namespace: kiste-system
  version: 0.3.0

spec:
  class: system
  role: control-plane

  interfaces:
    inputs:
      unitDefinitions:
        type: stream

      lifecycleRequests:
        type: event

      identityAssertions:
        type: identity.reference

      policyDecisions:
        type: stream

    outputs:
      plans:
        type: plan
        immutable: true

      lifecycleEvents:
        type: event

      evidence:
        type: evidence.bundle
        immutable: true

  capabilities:
    provides:
      - id: unit.resolve
      - id: unit.graph.resolve
      - id: capability.match
      - id: lifecycle.coordinate
      - id: execution.schedule
      - id: state.reconcile
      - id: failure.contain

    requires:
      - id: git.read
      - id: identity.authenticate
      - id: policy.evaluate
      - id: state.read
      - id: state.write
      - id: evidence.append
      - id: evidence.verify
      - id: execution.run

    conflicts:
      - id: evidence.rewrite
      - id: key.export
      - id: approval.self-grant

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

      - action: evidence.append
        resourceRef: kiste-evidence

    grants: []

  lifecycle:
    workflow: kiste.standard/v1

    steps:
      read:
        requireImmutableGitResolution: true

      inspect:
        checks:
          - source-integrity
          - capability-satisfaction
          - permissions
          - state-consistency
          - provider-health
          - recovery-readiness

      plan:
        requirePermissionDiff: true
        requireRecoveryStrategy: true
        produceImmutablePlan: true

      recommend:
        requirePolicyEvaluation: true

      apply:
        requireApproval: true
        requireExecutionLease: true
        useTwoPhaseApply: true

      recheck:
        verify:
          - desired-state
          - source-revision
          - permissions
          - security
          - health
          - performance
          - evidence-completeness

  safety:
    defaultMutationPolicy: deny

    safeMode:
      allow:
        - read
        - inspect
        - diagnostics
      deny:
        - apply
        - delete
        - permission-delegate
        - key-export

    authorization:
      defaultEffect: deny
      unavailablePolicyEngine: deny-mutations
      unverifiableIdentity: deny

    executionLease:
      required: true
      duration: 15m

    locking:
      mode: lease
      conflictAction: stop

    circuitBreaker:
      enabled: true
      failureThreshold: 3

    apply:
      immutablePlan: required
      twoPhase: required
      rollbackOrContainment: required

    recheck:
      required: true
      failureAction: rollback-or-contain

    separationOfDuties:
      selfApproval: deny
      selfEvidenceRewrite: deny

    breakGlass:
      enabled: true
      maximumDuration: 30m
      requireReason: true
      recordAllActions: true

  policies:
    git:
      immutableApplyRevision: required
      forcePush: deny
      signedCommits:
        requiredFor:
          - production

    permissions:
      defaultEffect: deny
      undeclaredAccess: deny

    audit:
      recordAllLifecycleSteps: true
      evidenceRequiredBeforeMutation: true
      appendOnlyEvidence: required

status:
  phase: Healthy

  conditions:
    - type: GitResolved
      status: "True"

    - type: IdentityAvailable
      status: "True"

    - type: PolicyAvailable
      status: "True"

    - type: StateStoreAvailable
      status: "True"

    - type: EvidenceStoreAvailable
      status: "True"

    - type: MutationAllowed
      status: "True"
```

---

## 15. Standard workload Unit with Git auditability

```yaml
apiVersion: kiste.dev/v1alpha1
kind: Unit

metadata:
  name: payment-api
  namespace: payments
  version: 1.2.0

spec:
  class: workload
  role: application

  sources:
    primary:
      type: git.repository
      role: definition

      location:
        providerRef: company-git
        repository: payments/payment-api
        path: .

      revision:
        branch: main

      audit:
        resolveToCommit: required
        recordTreeDigest: true
        requireProtectedRefFor:
          - production
        requireSignedCommitFor:
          - production

  interfaces:
    inputs:
      database:
        type: database.connection
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
      - id: database.connection
        version: "^1.0"

      - id: container.build
        version: "^1.0"

      - id: deployment.apply
        version: "^1.0"

  permissions:
    requests:
      - action: source.read
        resourceRef: primary

      - action: database.connect
        resourceRef: payment-database

      - action: deployment.apply
        resourceRef: production-runtime
        conditions:
          lifecycleStep: apply
          approval: required

  lifecycle:
    workflow: kiste.standard/v1

    steps:
      read:
        resolveGitRevision: true

      inspect:
        checks:
          - source-integrity
          - dependency-integrity
          - permissions
          - security
          - infrastructure
          - runtime

      plan:
        bindToResolvedCommit: true
        requireRollbackPlan: true

      recommend:
        requireApprovalFor:
          - production

      apply:
        useApprovedPlanOnly: true

      recheck:
        verify:
          - deployed-commit
          - desired-state
          - health
          - permissions
          - evidence

  policies:
    git:
      forcePush: deny
      immutableApplyRevision: required

    permissions:
      defaultEffect: deny

    audit:
      evidenceRequired: true
      bindEvidenceToGitCommit: true
```

---

## 16. Final project rules

### Unit rule

The Kiste Unit is the smallest complete lifecycle and permission boundary.

### Git rule

Every meaningful Unit revision should be traceable to an immutable Git commit or another content-addressed source revision.

### Audit rule

Git records declared history; immutable evidence records executed history.

### Lifecycle rule

Every Unit follows:

```text
Read → Inspect → Plan → Recommend → Apply → Recheck
```

### Apply rule

An apply action is valid only for the exact Git revision, Unit digest, plan, permissions, policies and approvals that were evaluated.

### Safety rule

When the Control Unit cannot establish certainty, it denies mutation and enters a safe state.

### Recovery rule

Every change must have rollback, compensation, containment or an explicit irreversible classification.

### Separation rule

The Control Unit may coordinate identity, policy, key management, execution and evidence, but it must not silently replace all of them with one unrestricted authority.

### Packaging rule

Control, identity, policy, key management and evidence may be deployed in one process for the MVP while remaining logically distinct system Units.

### Evidence rule

The Control Unit must not be capable of silently approving its own sensitive changes and rewriting the evidence proving those changes occurred.

---

## Safety chain

```text
Git commit
   ↓
Read and resolve
   ↓
Inspect
   ↓
Immutable plan
   ↓
Independent approval
   ↓
Leased two-phase apply
   ↓
Recheck
   ↓
Append-only evidence
```

This makes Git central to Kiste auditability without incorrectly treating Git alone as an unchangeable audit ledger.
