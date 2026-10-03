# Kiste v0.9.13C — LifecycleRun, Evidence Transactions, and Replay

**Status:** Draft architecture and implementation contract  
**Release:** `0.9.13C`  
**Depends on:** Kiste v0.9.13A and v0.9.13B  
**Theme:** Make Read → Inspect → Plan → Review → Deploy → Monitor one evidence-bound, replayable lifecycle transaction.

---

## 1. Purpose

Phase 9.13A defines six canonical Control Units:

```text
Read
Inspect
Plan
Review
Deploy
Monitor
```

Phase 9.13B establishes their trusted local runtime through:

```python
workspace, status = kiste.init()
```

Phase 9.13C defines how those Control Units cooperate during a lifecycle execution.

The central object introduced by this phase is:

```text
LifecycleRun
```

A LifecycleRun provides:

```text
one execution identity
immutable stage handoffs
evidence provenance
digest binding
approval binding
retry/resume
restart-by-replay
fail-closed transitions
safe Monitor re-entry
historical continuity for Model CV
```

The canonical flow becomes:

```text
Trusted Workspace
       ↓
LifecycleRun
       ↓
Read
       ↓
Inspect
       ↓
Plan
       ↓
Review
       ↓
Deploy
       ↓
Monitor
       ↓
Historical Evidence
```

---

## 2. Normative Architecture Decision

A lifecycle execution MUST be represented as one `LifecycleRun`.

A LifecycleRun is not a seventh Control Unit.

```text
LifecycleRun = transaction/evidence boundary

Read/Inspect/Plan/Review/Deploy/Monitor
             = Control Units
```

The LifecycleRun coordinates identity, stage handoff, evidence and state continuity.

It does not absorb the semantic responsibility of any Control Unit.

---

## 3. Execution Identity

Every LifecycleRun MUST receive a stable:

```text
executionId
```

Example:

```text
lifecycle-01JABC...
```

The same execution ID follows the run through all six stages:

```text
lifecycle-01JABC
├── Read
├── Inspect
├── Plan
├── Review
├── Deploy
└── Monitor
```

The execution ID MUST NOT be reused for a logically different lifecycle.

A new execution MUST be created when a change invalidates the assumptions of the existing run.

Examples:

```text
new source revision
changed intent
changed policy
changed trusted Unit set
changed approved plan
Monitor-triggered lifecycle re-entry
explicit user restart from a changed baseline
```

---

## 4. LifecycleRun

Conceptually:

```yaml
schema: kiste.lifecycle-run/v0.9.13c

executionId: lifecycle-01J...

workspace:
  ref: workspace://...
  controlDigest: sha256:...

createdAt: ...

state: running

baseline:
  configDigest: sha256:...
  intentDigest: sha256:...
  policyDigest: sha256:...
  controlRegistryDigest: sha256:...
  unitLockDigest: sha256:...

stages:
  read: ...
  inspect: ...
  plan: ...
  review: ...
  deploy: ...
  monitor: ...

parentExecutionId: null
triggerEvidenceRef: null
```

The LifecycleRun MUST contain references and digests rather than copying arbitrary mutable state.

---

## 5. Run State

A LifecycleRun has one of the following high-level states:

```text
created
running
blocked
completed
failed
superseded
```

`blocked` means Kiste understands why progression cannot continue.

Examples:

```text
missing evidence
stale evidence
unresolved capability
policy denial
missing approval
digest mismatch
changed target state
```

`failed` is reserved for lifecycle execution failure that cannot currently progress normally.

`superseded` means another LifecycleRun has replaced the run as the current decision path.

No state change may erase previous evidence.

---

## 6. Stage Attempts

A stage MAY have multiple attempts.

Each attempt MUST receive an:

```text
attemptId
```

Example:

```text
lifecycle-01J.../inspect/attempt-0001
lifecycle-01J.../inspect/attempt-0002
```

Retries MUST NOT overwrite earlier attempts.

```text
Inspect
├── attempt-0001 → fail
└── attempt-0002 → pass
```

A downstream stage MUST reference the exact upstream attempt it consumed.

Once a downstream stage begins from an accepted stage result, that upstream result is frozen for that lifecycle path.

If an already-consumed stage must be recomputed because its inputs changed, Kiste MUST create a new LifecycleRun.

---

## 7. Common StageResult

Every Control Unit MUST return a normalized `StageResult`.

Conceptually:

```yaml
schema: kiste.stage-result/v0.9.13c

executionId: lifecycle-01J...
attemptId: attempt-0001

stage: inspect

controlUnit:
  unitRef: kiste-system/inspect

status: pass

startedAt: ...
completedAt: ...

inputs:
  - ref: evidence://read/static/...
    digest: sha256:...

checks: []
findings: []

evidenceRefs:
  - evidence://inspect/runtime/...

outputs:
  requiredCapabilityGraph:
    ref: artifact://capabilities/required/...
    digest: sha256:...

nextActions:
  - plan
```

Canonical result statuses are:

```text
pass
warn
fail
blocked
```

`fail` and `blocked` MUST NOT automatically progress to the next stage.

`warn` MAY progress only when no blocking requirement or policy prevents it.

---

## 8. StageResult Is the Handoff Boundary

Control Units MUST communicate lifecycle state through durable StageResults and evidence references.

A later Control Unit MUST NOT depend on important hidden in-memory state from an earlier Control Unit.

Canonical rule:

```text
anything required for lifecycle continuation
must be representable by durable reference + digest
```

This allows:

```text
process restart
Control Unit replacement
local → gRPC migration
remote execution
replay
audit
Model CV reconstruction
```

without changing lifecycle semantics.

---

## 9. Evidence Record

9.13C standardizes a common evidence envelope.

Conceptually:

```yaml
schema: kiste.evidence/v0.9.13c

id: evidence-01J...

executionId: lifecycle-01J...
attemptId: attempt-0001

stage: inspect

producer:
  unitRef: kiste-system/inspect
  workerUnitRef: runtime/kubernetes-inspector

subject:
  ref: unit://models/example

classification: dynamic
verification: observed

generatedAt: ...
observedAt: ...

freshness:
  staleAfter: ...

payloadRef: blob://sha256/...

digest: sha256:...

confidence: 1.0
```

---

## 10. Evidence Classification

Evidence SHOULD use the following semantic classifications:

```text
declared
static
dynamic
resolution
approval
execution
historical
```

Examples:

```text
declared
    user intent

static
    source digest
    SBOM
    SAST
    dependency analysis

dynamic
    runtime probe
    hardware observation
    DAST

resolution
    selected KisteUnit
    capability resolution

approval
    review decision
    policy decision

execution
    deployment result
    mutation record

historical
    health
    drift
    cost
    incident
```

Evidence SHOULD separately describe whether a claim is:

```text
claimed
inferred
verified
observed
```

Inference MUST NOT silently become verified fact.

---

## 11. Evidence Is Append-Only

Lifecycle evidence MUST be:

```text
append
or
reference
```

not silently rewritten.

Example:

```text
Read:
    CUDA compatibility inferred

Inspect:
    CUDA runtime initialization failed
```

Kiste MUST preserve both records.

Inspect does not rewrite the Read evidence.

The Model CV may later display:

```text
earlier inference contradicted by runtime observation
```

but the original evidence remains part of history.

---

## 12. Evidence Index

The evidence index initialized by 9.13B becomes the central local evidence index.

Suggested structure:

```yaml
schema: kiste.evidence-index/v0.9.13c

executions:

  lifecycle-01J...:

    read:
      - evidence://...

    inspect:
      - evidence://...

    plan:
      - evidence://...

    review:
      - evidence://...

    deploy:
      - evidence://...

    monitor:
      - evidence://...
```

The index is not itself the evidence payload.

It is an index of immutable or content-addressed evidence.

---

## 13. Run Baseline

At creation, a LifecycleRun MUST bind the trusted baseline available at that time.

At minimum:

```text
WorkspaceControl digest
kiste.yaml/config digest
intent digest
policy digest
Control Unit registry digest
Unit lock/registry digest
requested source references
```

Read later resolves source references into immutable source identity.

Example:

```text
requested:
    branch main

resolved:
    commit abc123
    tree sha256:...
```

After Read accepts this resolution, downstream stages MUST use that resolved source.

---

## 14. Progressive Binding

Some lifecycle facts cannot exist when the run begins.

The run therefore accumulates immutable bindings as stages complete.

Conceptually:

```text
Run creation
    config
    intent
    policy
    control registry

Read
    resolved source
    static evidence

Inspect
    observed state
    dynamic evidence
    RequiredCapabilityGraph

Plan
    ResolvedCapabilityGraph
    ExecutionPlan
    PermissionPlan
    RollbackPlan

Review
    ReviewDecision
    ApprovalEvidence

Deploy
    execution result
    changed resources

Monitor
    runtime history
```

Each accepted binding MUST be represented by reference and digest.

---

## 15. Read Contract

Read consumes:

```text
trusted WorkspaceControl
declared intent
source references
Unit definitions
policy references
previous evidence where explicitly referenced
```

Read produces:

```text
ResolvedSourceSet
StaticEvidence
DeclaredCapabilitySet
StaticDiscoveredCapabilitySet
Read StageResult
```

Read MUST bind mutable source references to immutable identifiers where possible.

Examples:

```text
Git branch → commit
container tag → digest
model revision → immutable revision
artifact → digest
```

Read remains non-mutating.

---

## 16. Inspect Contract

Inspect consumes:

```text
accepted Read StageResult
ResolvedSourceSet
StaticEvidence
intent
policy
allowed observation context
```

Inspect produces:

```text
DynamicEvidence
ObservedState
DiscoveredCapabilitySet
RequiredCapabilityGraph
MissingEvidenceReport
CapabilityConflictReport
PlanningReadinessDecision
Inspect StageResult
```

Inspect MUST reference the exact Read result it consumed.

Inspect MUST NOT choose final implementation bindings.

Inspect remains non-mutating with respect to managed target infrastructure.

Controlled probes MAY run only under explicit inspection authority.

---

## 17. Plan Contract

Plan consumes:

```text
accepted Read result
accepted Inspect result
RequiredCapabilityGraph
available KisteUnits
policy
trust constraints
preferences
observed state
```

Plan produces:

```text
CandidateImplementationGraph
RejectedImplementationReport
ResolvedCapabilityGraph
ActionGraph
ExecutionPlan
PermissionPlan
RollbackPlan
ResolutionEvidence
Plan StageResult
```

Plan MUST bind its result to:

```text
source digest
intent digest
policy digest
required capability graph digest
observed-state digest where relevant
Unit registry/lock digest
```

Plan is non-mutating.

---

## 18. Review Contract

Review consumes the exact accepted Plan result.

Review produces:

```text
ReviewDecision
ApprovalEvidence
PolicyEvidence
ApprovedPlan
or
RejectedPlan
Review StageResult
```

Review does not execute the plan.

An approval MUST bind to the exact reviewed plan and relevant lifecycle baseline.

Conceptually:

```yaml
approval:
  executionId: lifecycle-01J...
  planDigest: sha256:...
  sourceDigest: sha256:...
  policyDigest: sha256:...
  resolvedCapabilityGraphDigest: sha256:...
  decision: approved
  recordedAt: ...
```

An approval for Plan A MUST NOT authorize Plan B.

---

## 19. Deploy Contract

Deploy consumes:

```text
accepted Plan result
accepted Review result
ApprovedPlan
current source identity
current policy identity
current target preconditions
execution authority
```

Before mutation, Deploy MUST revalidate:

```text
approved plan digest
source digest
policy digest
Unit/implementation digest
approval digest
required execution permissions
target state/preconditions where applicable
```

Mismatch MUST block mutation.

Deploy produces:

```text
ExecutionResult
ExecutionEvidence
ChangedResourceReferences
RollbackResult where applicable
Deploy StageResult
```

Deploy MUST NOT infer approval from `kiste init`.

Deploy MUST NOT use ambient mutation authority merely because the Deploy Control Unit exists.

---

## 20. Deploy Idempotency

Deploy MUST be designed for at-least-once invocation.

A deployment attempt SHOULD have an idempotency key derived from:

```text
executionId
attemptId
approved plan digest
target identity
```

Repeated invocation of the same deployment attempt MUST NOT silently create additional unintended mutations.

Conflicting duplicate results MUST fail closed.

---

## 21. Monitor Contract

Monitor consumes:

```text
Deploy result
expected deployed state
execution evidence
runtime references
policy
previous Monitor evidence
```

Monitor produces:

```text
HealthEvidence
DriftEvidence
PerformanceEvidence
SecurityEvidence
CostEvidence
HistoricalEvidence
LifecycleReentryRecommendation
Monitor StageResult
```

Monitor MUST NOT silently mutate the desired state.

Monitor may observe repeatedly within one LifecycleRun.

Each observation is a new attempt/evidence record.

---

## 22. Monitor Re-entry

Monitor may discover:

```text
source change
configuration drift
security incident
runtime incompatibility
cost threshold
performance regression
target-state drift
new policy
```

Monitor MUST NOT rewrite earlier lifecycle history.

Instead it may recommend a new lifecycle execution.

Example:

```text
LifecycleRun A
    ↓
Monitor detects runtime drift
    ↓
LifecycleRun B
    parentExecutionId = A
    triggerEvidenceRef = drift evidence
```

Depending on the reason, the new lifecycle may conceptually re-enter through:

```text
Read
or
Inspect
```

but it is still a new LifecycleRun.

---

## 23. Lifecycle Lineage

New runs SHOULD retain lineage:

```yaml
executionId: lifecycle-B

parentExecutionId: lifecycle-A

trigger:
  type: monitor-reentry
  evidenceRef: evidence://monitor/drift/...
```

This creates history such as:

```text
Run A
  initial deployment
     ↓
Run B
  drift correction
     ↓
Run C
  source revision
     ↓
Run D
  policy-driven migration
```

This history feeds the Model CV.

---

## 24. Freshness

Dynamic evidence may become stale.

Evidence MAY declare:

```text
observedAt
staleAfter
```

Before consuming dynamic evidence, a stage MUST evaluate freshness when freshness is relevant to safety or correctness.

Example:

```text
GPU availability observed 3 days ago
```

may not be sufficient evidence for a deployment requiring current GPU capacity.

Stale required evidence MUST result in:

```text
blocked
```

or a new inspection.

---

## 25. Static Evidence Invalidation

Static evidence is primarily bound to source identity.

When:

```text
source digest changes
```

static evidence produced against the previous source MUST NOT automatically be treated as current evidence.

A new Read run is normally required.

---

## 26. Plan Invalidation

A plan may become invalid when any bound prerequisite changes.

Examples:

```text
source changed
intent changed
policy changed
RequiredCapabilityGraph changed
resolved Unit changed
critical observed state changed
permission assumptions changed
```

A changed prerequisite MUST NOT be compensated for by merely updating the plan digest.

The plan must be regenerated and reviewed.

---

## 27. Approval Invalidation

Approval is invalid if its bound context no longer matches.

Examples:

```text
plan digest mismatch
source digest mismatch
policy digest mismatch
approval expiration where used
required capability graph mismatch
resolved implementation mismatch
```

Deploy MUST fail closed.

---

## 28. Retry

Retry is allowed when the lifecycle baseline remains valid.

Example:

```text
Inspect attempt 1
    temporary runtime timeout
        ↓
Inspect attempt 2
    pass
```

Both attempts remain auditable.

A retry MUST NOT overwrite the failed attempt.

---

## 29. Resume

A LifecycleRun MUST be resumable after process restart.

Kiste SHOULD be able to reconstruct the run from durable artifacts without relying on process memory.

Conceptually:

```text
restart Kiste
    ↓
load WorkspaceControl
    ↓
load LifecycleRun
    ↓
verify digests
    ↓
find last accepted stage
    ↓
resume
```

A resumed run MUST revalidate relevant prerequisites before continuing.

---

## 30. Restart-by-Replay

Lifecycle coordination MUST be reproducible from persisted facts.

Kiste MUST NOT require hidden controller memory to determine:

```text
which stage ran
which attempt succeeded
which evidence was produced
which plan was reviewed
which approval was granted
which deployment occurred
```

Implementations SHOULD maintain an ordered lifecycle journal.

Suggested:

```text
.kiste/runs/<executionId>/events.jsonl
```

Possible facts:

```text
run.created
stage.started
evidence.recorded
stage.completed
review.approved
deploy.started
deploy.completed
monitor.observed
run.blocked
run.completed
```

The journal records operational facts.

It does not become the desired-state source of truth.

---

## 31. Git and Desired State

LifecycleRun does not replace Git or declared intent.

Canonical rule:

```text
desired state = declared intent

observed state = evidence

plan = transition

LifecycleRun = evidence-bound history of that transition
```

Git remains the primary declared-source history where Git is used.

The LifecycleRun records what Kiste observed and did.

Runtime observations MUST NOT silently rewrite user intent or source history.

---

## 32. Stage Transition Rules

The normal transition graph is:

```text
Read
  ↓
Inspect
  ↓
Plan
  ↓
Review
  ↓
Deploy
  ↓
Monitor
```

A transition may occur only when the required upstream result is accepted.

Canonical requirements:

```text
Inspect requires accepted Read.

Plan requires accepted Read + Inspect.

Review requires accepted Plan.

Deploy requires approved Review + exact Plan.

Monitor requires an execution baseline.
```

No Control Unit may silently skip a required safety boundary.

---

## 33. Missing Evidence

Required evidence that cannot be established MUST be explicit.

Example:

```yaml
status: blocked

findings:
  - type: missing-evidence
    requirement: gpu.driver.compatibility
```

Missing evidence is not equivalent to:

```text
false
safe
unsupported
approved
```

Unknown remains unknown.

If the unknown affects mutation safety, Kiste MUST fail closed.

---

## 34. Capability Resolution Safety

9.13C preserves the 9.13A capability model:

```text
RequiredCapabilityGraph
        ↓
Plan
        ↓
ResolvedCapabilityGraph
```

Unknown required capabilities MUST block deployment.

A hard capability constraint cannot be overcome by preference scoring.

Policy always overrides preference.

---

## 35. Secret Handling

Lifecycle evidence SHOULD contain secret references rather than secret values.

Example:

```text
vault://...
aws-sm://...
env://...
```

If a Control Unit requires secret-value access, that access MUST be:

```text
stage-specific
policy-authorized
minimum-scope
auditable
```

Secret values MUST NOT be placed into ordinary StageResults, evidence indexes, run journals or Model CV output.

---

## 36. Concurrency

Within one LifecycleRun, stage progression is logically serialized.

Different LifecycleRuns MAY coexist.

Non-mutating work MAY execute concurrently when their inputs are independently pinned.

Mutating Deploy operations MUST use target-scoped concurrency protection.

Two LifecycleRuns MUST NOT concurrently mutate the same protected target unless the backend and policy explicitly support such behavior.

---

## 37. Target Locking

Before mutation, Deploy SHOULD acquire an execution/target lock appropriate to the backend.

The lock MUST NOT replace digest revalidation.

Canonical sequence:

```text
approval valid
      ↓
acquire target lock
      ↓
revalidate state
      ↓
execute exact approved plan
      ↓
record evidence
      ↓
release lock
```

---

## 38. Adapter Independence

LifecycleRun semantics must not depend on how a Control Unit is packaged.

The same StageResult/evidence model applies to:

```text
embedded
native
local gRPC
remote gRPC
WASM
```

A remote Inspect implementation and an embedded Inspect implementation must produce semantically compatible outputs.

---

## 39. Worker KisteUnits

Control Units may call worker KisteUnits.

Example:

```text
Read
├── SAST Unit
├── SCA Unit
└── SBOM Unit

Inspect
├── runtime probe Unit
├── DAST Unit
└── hardware Unit

Plan
├── placement Unit
├── cost Unit
└── backend-planning Unit
```

Worker-native results MUST be normalized into:

```text
evidence
findings
capability contributions
measurements
artifacts
```

before becoming lifecycle facts.

9.13C does not require specific third-party integrations.

---

## 40. Model CV Integration

The Model CV MUST be derived from lifecycle evidence.

It MUST NOT maintain a competing independent history.

Conceptually:

```text
LifecycleRun A
LifecycleRun B
LifecycleRun C
       ↓
Evidence Index
       ↓
Model CV
```

Every significant Model CV claim SHOULD reference:

```text
executionId
stage
evidenceRef
digest
producer
time
verification state
```

This allows Kiste to answer:

```text
When was this verified?

Against which source revision?

By which Unit?

Was it inferred or observed?

Which plan used it?

Who approved the plan?

What was actually deployed?

Did later evidence contradict it?
```

---

## 41. Contradictory Evidence

Contradictory evidence MUST be preserved.

Example:

```text
Run A:
    model works on L40S

Run C:
    same model/version fails after driver change
```

Kiste should represent the temporal/context difference.

It MUST NOT delete the earlier evidence merely because newer evidence differs.

---

## 42. Suggested Artifact Layout

```text
.kiste/

  runs/
    lifecycle-01J.../
      run.json
      bindings.json
      events.jsonl

      read/
        attempt-0001.json

      inspect/
        attempt-0001.json
        attempt-0002.json

      plan/
        attempt-0001.json

      review/
        attempt-0001.json

      deploy/
        attempt-0001.json

      monitor/
        attempt-0001.json
        attempt-0002.json

  evidence/
    index.json
    ...

  capabilities/
    ...

  plans/
    ...

  reviews/
    ...

  deploy/
    ...

  monitor/
    ...

  cv/
    ...
```

Exact paths remain implementation details unless separately standardized.

The lifecycle relationships and digest bindings are normative.

---

## 43. Public API Direction

9.13C MUST expose LifecycleRun identity through the programming API.

A possible API is:

```python
workspace, status = kiste.init()

read_result = kiste.read(workspace)

execution_id = read_result.execution_id

inspect_result = kiste.inspect(
    workspace,
    execution_id=execution_id,
)
```

The exact convenience API MAY evolve.

The normative requirement is:

```text
every lifecycle stage knows which LifecycleRun it belongs to
```

and every StageResult exposes its `executionId`.

CLI commands MUST similarly expose the execution identity.

---

## 44. Active Run Convenience

Implementations MAY maintain a convenience reference to the currently resumable lifecycle run.

This reference:

```text
MUST NOT be authority
MUST NOT replace executionId
MUST resolve unambiguously
MUST be safe to reconstruct
```

If multiple candidate runs exist and the intended run is ambiguous, the user or calling program MUST specify the execution ID.

---

## 45. Failure Semantics

Normal lifecycle problems SHOULD be represented through StageResult rather than uncontrolled process termination.

Examples:

```text
missing evidence
policy denial
capability conflict
stale approval
source mismatch
target drift
```

should normally yield:

```text
status = blocked
or
status = fail
```

Internal implementation faults may raise exceptions at the programming boundary, but any durable facts produced before the fault MUST remain preserved.

---

## 46. Audit Rule

Every consequential lifecycle decision MUST be traceable.

The minimum chain is:

```text
decision
   ↓
StageResult
   ↓
evidence reference
   ↓
executionId
   ↓
Control Unit
   ↓
worker Unit/tool where applicable
   ↓
source or runtime observation
```

The central audit rule is:

> A later stage must be able to prove exactly which earlier evidence and result it consumed.

---

## 47. Security Rules

The following are normative:

```text
Read is non-mutating.

Inspect does not mutate managed infrastructure.

Plan is non-mutating.

Review does not execute.

Deploy executes only an approved immutable plan.

Monitor does not silently mutate desired state.

Unknown required capability blocks mutation.

Missing required evidence blocks mutation.

Stale safety-critical evidence blocks mutation.

Source mismatch invalidates downstream assumptions.

Policy mismatch invalidates downstream assumptions.

Plan mismatch invalidates approval.

Approval mismatch blocks Deploy.

Evidence is not silently rewritten.

Retries preserve earlier attempts.

Re-entry creates new lifecycle history.

Deploy uses explicit scoped authority.

Secret values are excluded from normal evidence.

Lifecycle state is restartable from durable facts.
```

---

## 48. Implementation Migration

Recommended `kiste-py` migration order:

```text
1. Add LifecycleRun model.

2. Add executionId generation and persistence.

3. Add StageResult model.

4. Add stage attempt identity.

5. Add common Evidence model.

6. Upgrade evidence index to execution-aware form.

7. Bind Read output to immutable source identity.

8. Make Inspect consume exact Read result.

9. Make Plan consume exact Inspect result.

10. Bind Review approval to exact Plan digest.

11. Make Deploy revalidate all approval bindings.

12. Add Monitor evidence and re-entry recommendation.

13. Add retry/resume semantics.

14. Add persistent run journal.

15. Add restart-by-replay tests.

16. Build Model CV from LifecycleRuns.
```

Existing stage implementations may initially remain embedded.

Semantic migration takes priority over physical decomposition.

---

## 49. Test Requirements

`kiste_core` MUST test at least:

```text
LifecycleRun receives stable executionId

all six stages preserve executionId

each stage references exact upstream StageResult

retry creates new attemptId without deleting prior attempt

process restart can resume a valid run

source digest change blocks stale Plan/Deploy

policy digest change invalidates approval

plan digest change invalidates approval

Deploy cannot execute without approved Review result

Deploy cannot execute when approval binds another plan

Deploy uses no init-derived mutation authority

missing required evidence blocks progression

stale required dynamic evidence blocks progression

unknown required capability blocks Deploy

evidence remains append-only

failed attempts remain auditable

Monitor observations append history

Monitor re-entry creates linked new LifecycleRun

parentExecutionId and triggerEvidenceRef are preserved

Model CV claims can resolve to lifecycle evidence

secret values are not written to normal run/evidence artifacts

duplicate Deploy invocation is idempotent or safely rejected

conflicting duplicate execution result fails closed

CLI and Python expose the same execution identity semantics
```

---

## 50. Dogfood Test

Kiste MUST be able to run the lifecycle against its own repository.

Conceptually:

```text
workspace, status = kiste.init()

assert workspace is not None

Read Kiste
     ↓
Inspect Kiste
     ↓
Plan Kiste
     ↓
Review plan
     ↓
Deploy only if explicitly authorized
     ↓
Monitor
```

The dogfood run MUST produce a complete evidence lineage.

A restart between stages SHOULD still allow the run to resume safely.

---

## 51. Non-Goals

Phase 9.13C does not require:

```text
Semgrep integration
Trivy integration
Checkov integration
ZAP integration
Tetragon integration
KubeVela execution integration
SkyPilot integration
Terraform/OpenTofu backend implementation
Slurm implementation
remote evidence database
distributed scheduler
six Control Unit services
TOSCA
Puccini
automatic remediation
automatic approval
universal optimization
```

Those may build on top of the LifecycleRun contract later.

---

## 52. Acceptance Criteria

Phase 9.13C is accepted when:

```text
1. Every lifecycle execution has one stable executionId.

2. LifecycleRun is not a seventh Control Unit.

3. Every stage produces a normalized StageResult.

4. Every stage attempt has a unique attemptId.

5. Retries preserve previous attempts.

6. Stage handoffs use durable references and digests.

7. Read binds mutable source references to immutable source identity.

8. Inspect consumes the exact accepted Read result.

9. Plan consumes exact Read/Inspect evidence.

10. Review approval binds to an exact Plan and lifecycle baseline.

11. Deploy verifies approval and prerequisite digests before mutation.

12. Changed source, policy or plan invalidates stale downstream authority.

13. Evidence is append-only/reference-based.

14. Missing required evidence blocks mutation.

15. Stale required evidence is identified and blocks where required.

16. Unknown required capability blocks mutation.

17. Lifecycle runs can survive process restart.

18. Run state is reconstructible from durable artifacts.

19. Monitor can append observations without rewriting history.

20. Monitor-triggered re-entry creates a linked new LifecycleRun.

21. Lifecycle lineage records parent execution and trigger evidence.

22. Deploy supports idempotent or safely rejectable repeated invocation.

23. Mutating execution uses target-scoped concurrency protection.

24. Secret values do not enter normal lifecycle evidence.

25. Model CV is derived from LifecycleRun evidence.

26. Contradictory historical evidence remains preserved.

27. Existing Control Units may remain embedded.

28. Adapter transport does not change lifecycle semantics.

29. CLI and programming APIs expose execution identity.

30. `kiste_core` tests enforce these transaction, replay,
    evidence and authorization rules.
```

---

## 53. Final Rule

Phase 9.13C establishes the rule:

```text
Nothing moves forward merely because the previous
function happened to run.

It moves forward because Kiste can prove:

    which lifecycle it belongs to,
    which exact result it consumed,
    which evidence supports that result,
    which source and policy were bound,
    which plan was approved,
    and whether those assumptions are still valid.
```

The lifecycle therefore becomes:

```text
Read
  evidence
    ↓
Inspect
  evidence
    ↓
Plan
  decision
    ↓
Review
  authorization
    ↓
Deploy
  execution
    ↓
Monitor
  history
    ↓
new LifecycleRun when reality changes
```

That is the transaction and audit backbone on which later Kiste tool integrations, multi-cloud placement, edge execution, robotics, HPC and Model CV features can safely build.
