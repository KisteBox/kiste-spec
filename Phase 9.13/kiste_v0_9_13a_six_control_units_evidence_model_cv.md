# Kiste v0.9.13A — Six Control Units, Evidence Pipeline, and Model CV Architecture

Status: Architecture correction and consolidation  
Release: `0.9.13A`  
API: `kiste.dev/v1alpha1` unless a later schema change requires otherwise  
Theme: Separate static knowledge, observed runtime evidence, capability synthesis, controlled execution, and accumulated system history into six canonical Control Units.

---

## 1. Purpose

Kiste has evolved from repository inspection and deployment-readiness tooling into a capability-first, Unit-centered control system.

The existing architecture already contains:

```text
KisteUnits
Capabilities
Workspace intent
Policy
Git history
Static inspection
Runtime observations
Capability graphs
Plans
Approvals
Deployment evidence
Monitoring evidence
```

Phase 9.13A consolidates these concepts into one canonical control-plane architecture.

The central flow is:

```text
Intent
  ↓
Read
  ↓
Static Evidence
  ↓
Inspect
  ↓
Dynamic Evidence + RequiredCapabilityGraph
  ↓
Plan
  ↓
ResolvedCapabilityGraph + Execution Plan
  ↓
Review
  ↓
Approved Plan
  ↓
Deploy
  ↓
Execution Evidence
  ↓
Monitor
  ↓
Observed History
  ↺
```

The accumulated evidence becomes the basis of a living **Model CV / System CV**.

Kiste's purpose is not to own every implementation. Its purpose is to understand a Unit well enough to determine what it is, what it needs, what evidence supports those conclusions, which implementations are allowed, what should change, what was approved, and what actually happened.

---

## 2. Normative Architecture Decision

Kiste has exactly six canonical Control Units:

```text
1. Read Control Unit
2. Inspect Control Unit
3. Plan Control Unit
4. Review Control Unit
5. Deploy Control Unit
6. Monitor Control Unit
```

These are not merely six function names inside one controller.

They are six logical, system-class Control Units with distinct responsibilities, inputs, outputs, permissions, and evidence boundaries.

An implementation may initially package several Control Units in one process or binary.

Physical packaging does not change the logical architecture.

```text
Logical separation is mandatory.
Physical separation is optional.
```

---

## 3. Canonical Lifecycle Vocabulary

Phase 9.13A standardizes the lifecycle as:

```text
Read
Inspect
Plan
Review
Deploy
Monitor
```

Earlier terminology maps as follows:

```text
Recommend → Review
Apply     → Deploy
Recheck   → Monitor
```

New specifications must use the canonical six names.

Existing documents using:

```text
Read
Inspect
Plan
Recommend
Apply
Recheck
```

are historical terminology and must not define a second lifecycle.

The semantic meaning is preserved:

```text
Recommend is review and approval behavior.
Apply is deployment and mutation behavior.
Recheck is monitoring and post-deployment verification behavior.
```

---

## 4. Control Units Are KisteUnits

The six Control Units follow the same fundamental KisteUnit model as other parts of Kiste.

Conceptually:

```text
KisteUnit
├── identity
├── inputs
├── outputs
├── capabilities
├── permissions
├── relationships
├── evidence
└── lifecycle participation
```

A Control Unit is therefore a privileged system-class KisteUnit.

Conceptual classification:

```yaml
spec:
  class: system
  role: control-unit
```

Control Units coordinate other KisteUnits.

They do not absorb every implementation into Core.

Example:

```text
Inspect Control Unit
   ├── runtime inspector Unit
   ├── DAST Unit
   ├── Kubernetes observer Unit
   ├── cloud-state Unit
   └── hardware inspector Unit
```

The Control Unit owns coordination and semantic output.

Worker KisteUnits own tool-specific behavior.

---

## 5. Read Control Unit

### 5.1 Purpose

Read establishes stable knowledge required for later reasoning.

Phase 9.13A defines Read as:

> Resolve immutable inputs and produce evidence that can be obtained without executing or actively probing the workload.

Read includes both resolution and static analysis.

```text
READ = source resolution + static evidence
```

### 5.2 Read Inputs

Read may consume:

```text
user intent
kiste.yaml
KisteUnit definitions
capability definitions
Git repositories
local directories
OCI metadata
model metadata
dependency manifests
infrastructure definitions
CI/CD definitions
policy references
credential references
existing lock files
previous Model CV evidence
```

### 5.3 Read Responsibilities

Read may perform:

```text
resolve source locations
resolve Git branches/tags to immutable revisions
calculate repository/tree digests
read Unit definitions
read capability declarations
read dependency manifests
read Dockerfiles
read Kubernetes manifests
read Terraform/IaC
read CI/CD configuration
read model configuration
read dataset references
read SBOM data
perform SAST
perform SCA
perform secret scanning
perform license analysis
perform static IaC analysis
infer static application properties
infer static capability evidence
load policy references
load previous evidence
```

Examples of tools that may participate through KisteUnits include:

```text
Semgrep
Trivy
Checkov
SBOM tooling
Git
OCI libraries
language dependency scanners
model metadata readers
```

These tools are not Core concepts.

They are implementations behind capabilities and KisteUnits.

### 5.4 Read Must Not

Read must not:

```text
execute the application
send DAST traffic
benchmark the workload
probe live service behavior
modify cloud infrastructure
modify Kubernetes
synthesize the deployment architecture
select the final capability implementation stack
deploy anything
```

### 5.5 Read Output

The primary output is a versioned static evidence bundle.

Conceptually:

```text
StaticEvidence
├── source identity
├── immutable revisions
├── code facts
├── dependency facts
├── artifact facts
├── infrastructure declarations
├── security findings
├── supply-chain findings
├── declared capabilities
├── static inferred capabilities
└── evidence digests
```

Suggested artifacts:

```text
.kiste/read/source-evidence.json
.kiste/read/static-evidence.json
.kiste/read/dependency-evidence.json
.kiste/read/security-static-evidence.json
.kiste/read/artifact-evidence.json
```

Exact filenames are implementation details unless separately standardized.

---

## 6. Inspect Control Unit

### 6.1 Purpose

Inspect discovers how the system actually behaves or currently exists.

Phase 9.13A defines Inspect as:

> Observe runtime, environment, infrastructure, security, hardware, and operational reality, then combine that evidence with Read output and intent to determine what capabilities are required.

Conceptually:

```text
INSPECT = dynamic observation + requirement assembly
```

### 6.2 Inspect Inputs

Inspect consumes:

```text
StaticEvidence
user intent
workspace policy
Unit graph
current environment references
read-only runtime/cloud access
previous observations
monitoring history where relevant
```

### 6.3 Inspect Responsibilities

Inspect may perform:

```text
runtime execution in an allowed sandbox
DAST
API probing
health probing
network observation
dependency behavior observation
container runtime inspection
Kubernetes state inspection
cloud resource observation
IAM observation
hardware detection
GPU/accelerator compatibility tests
model loading tests
inference probes
latency measurement
memory observation
runtime security observation
existing infrastructure discovery
drift observation
environment compatibility checks
```

Examples of participating tools may include:

```text
OWASP ZAP
Tetragon
Kubernetes API
cloud APIs
container runtime APIs
GPU/runtime probes
model inference probes
runtime benchmark tools
telemetry systems
```

The tool list is non-normative. Capability contracts remain the stable interface.

### 6.4 Static and Dynamic Security Are Separate

Security scanning is not owned exclusively by one Control Unit.

Placement depends on how evidence is obtained.

```text
Static:
    Read

Examples:
    SAST
    SCA
    SBOM
    IaC analysis
    secret scanning

Dynamic:
    Inspect

Examples:
    DAST
    runtime network observation
    runtime process/syscall observation
    identity behavior
    effective runtime permissions
```

The lifecycle boundary is based on evidence semantics, not the word `security`.

---

## 7. Capability Requirement Assembly

Capability requirement assembly remains an Inspect responsibility.

Inspect answers:

```text
What capabilities are required?
```

It does not answer:

```text
Which implementation should provide them?
```

Conceptually:

```text
RequiredCapabilityGraph = assemble(
    intent,
    staticEvidence,
    dynamicEvidence,
    UnitDeclarations,
    policyConstraints,
    environmentConstraints
)
```

Inputs include:

```text
declared capabilities
static inferred capabilities
dynamic observations
existing infrastructure
current runtime state
policy-added requirements
environment constraints
Unit dependencies
```

Every inferred requirement must preserve:

```text
source
evidence
confidence
reason
timestamp where applicable
digest
```

### 7.1 Inspect Output

Inspect produces:

```text
DynamicEvidence
DeclaredCapabilitySet
DiscoveredCapabilitySet
RequiredCapabilityGraph
MissingEvidenceReport
CapabilityConflictReport
InspectionEvidenceBundle
PlanningReadinessDecision
```

Conceptual artifact:

```yaml
apiVersion: kiste.dev/v1alpha1
kind: InspectionResult

metadata:
  name: payment-api-inspection

spec:
  intentDigest: sha256:...
  staticEvidenceDigest: sha256:...
  unitGraphDigest: sha256:...
  policyDigest: sha256:...

status:
  observations:
    runtime: []
    infrastructure: []
    security: []
    network: []
    hardware: []
    performance: []

  capabilities:
    declared: []
    discovered: []

    requiredGraph:
      nodes: []
      edges: []

  missingEvidence: []
  conflicts: []

  planningReady: true
```

### 7.2 Planning Readiness

`planningReady` must be false when:

```text
required evidence is unavailable
runtime observations are stale
source revision changed
policy cannot be evaluated
required capability identity is unknown
Unit graph resolution failed
required capability dependencies cannot resolve
hard constraints conflict
```

Planning must not silently invent missing evidence.

---

## 8. Plan Control Unit

### 8.1 Purpose

Plan determines how required capabilities should be satisfied and what change should occur.

Phase 9.13A explicitly places **capability synthesis inside the Plan Control Unit**.

There is no seventh lifecycle phase named Capability Synthesis.

Correct architecture:

```text
Inspect Control Unit
    ↓
RequiredCapabilityGraph
    ↓
Plan Control Unit
    ├── capability synthesis
    ├── implementation discovery
    ├── compatibility filtering
    ├── trust filtering
    ├── policy filtering
    ├── deal-breaker filtering
    ├── preference ranking
    ├── graph optimization
    ├── topology generation
    └── change planning
```

### 8.2 Plan Answers Two Questions

First:

```text
What should provide the required capabilities?
```

Second:

```text
What should change?
```

Therefore:

```text
Inspect determines requirements.
Plan resolves implementations and actions.
```

### 8.3 Plan Responsibilities

Plan may perform:

```text
expand capability dependency graph
discover candidate KisteUnits
discover compatible tools
apply technical compatibility rules
apply policy
apply trust constraints
apply deal breakers
apply explicit bindings
rank preferences
optimize Unit selection
minimize permissions
minimize unnecessary Units
produce ResolvedCapabilityGraph
generate deployment topology
generate execution order
calculate permissions
calculate expected resource changes
calculate cost implications
calculate rollback/containment
generate backend-specific plan
```

### 8.4 Deterministic Resolution

Hard gates precede preference.

Canonical order:

```text
technical compatibility
    ↓
organization policy
    ↓
trust boundary
    ↓
user deal breakers
    ↓
explicit bindings
    ↓
capability-specific preference
    ↓
intent preference
    ↓
fewest permissions / fewest Units
    ↓
fail on unresolved tie
```

No numeric score may compensate for a failed hard constraint.

### 8.5 Plan Output

Plan produces:

```text
ResolvedCapabilityGraph
SelectedUnitGraph
RejectedCandidateReport
ResolutionEvidence
Topology
ActionGraph
ExecutionPlan
PermissionPlan
RollbackPlan
CostEstimate
PlanDigest
```

No mutation occurs during Plan.

---

## 9. Plan Representation and Compilation Boundary

Kiste owns the semantic planning model.

```text
RequiredCapabilityGraph
        ↓
Capability synthesis
        ↓
ResolvedCapabilityGraph
        ↓
ExecutionPlan
        ↓
Backend adapter
```

The Plan Control Unit must not depend on a single external topology language, compiler, cloud API, or orchestration engine.

Backend-specific representations are generated after Kiste has completed capability resolution.

Conceptually:

```text
ResolvedCapabilityGraph
        ↓
ExecutionPlan
        ├── KubeVela/OAM
        ├── Kubernetes
        ├── SkyPilot
        ├── Terraform/OpenTofu
        ├── Slurm/HPC
        └── future backends
```

For the current implementation direction:

```text
Kiste model
    ↓
KubeVela/OAM
    ↓
Kubernetes
```

KubeVela/OAM is the default implementation target for Kubernetes application delivery where appropriate, but it does not define Kiste's internal capability, evidence, Unit, or Model CV models.

Portable topology standards and compiler interfaces may be added in a future release when those ecosystems are sufficiently mature.

Phase 9.13A does **not** standardize TOSCA integration, TOSCA compilation, Puccini integration, Clout, or Floria. These are deferred compatibility concerns.

---

## 10. Review Control Unit

Review decides whether the plan is justified and authorized.

Review consumes:

```text
StaticEvidence
DynamicEvidence
RequiredCapabilityGraph
ResolvedCapabilityGraph
ExecutionPlan
policy
risk evidence
permission requirements
selection reasoning
rollback strategy
```

Review responsibilities include:

```text
policy validation
security review
permission review
risk review
capability resolution validation
plan validation
evidence completeness
change-boundary validation
human approval
automated approval policy
partial approval
rejection
change request
risk acceptance
```

Review must be independent of Plan's preference ranking.

A preferred implementation is not automatically an approved implementation.

---

## 11. Deploy Control Unit

Deploy executes only an approved immutable plan.

Canonical rule:

```text
No approved plan → no mutation.
```

Deploy must verify before execution:

```text
plan digest
source revision
Unit graph digest
policy digest
capability resolution
approval record
permission set
target state assumptions
execution lease where required
```

If material inputs changed after review:

```text
Deploy stops.
```

Deploy may invoke KisteUnits backed by:

```text
KubeVela
Kubernetes
GitOps systems
cloud APIs
SkyPilot
Terraform/OpenTofu
Ansible
Slurm
edge runtimes
other execution backends
```

The backend does not become the Kiste control model.

---

## 12. Monitor Control Unit

Monitor continuously compares expected state with observed reality.

Monitor responsibilities include:

```text
health
availability
performance
cost
security
policy compliance
capability availability
runtime drift
infrastructure drift
deployment drift
model performance
resource use
accelerator behavior
version drift
dependency drift
evidence freshness
```

Monitor produces historical evidence.

When significant drift or failure occurs:

```text
Monitor
   ↓
new observation
   ↓
Inspect
```

Where source or configuration changed:

```text
Monitor
   ↓
Read
```

The lifecycle is therefore a control loop rather than a one-time pipeline.

---

## 13. Evidence Model

Phase 9.13A defines four primary evidence classes for the Model CV.

### 13.1 Declared Evidence

What a user, author, Unit, or policy claims.

Examples:

```text
intent
declared requirements
expected runtime
claimed compatibility
declared resources
declared policies
```

### 13.2 Static Evidence

What Kiste can establish without executing the workload.

Produced primarily by Read.

Examples:

```text
Git commit
dependencies
SBOM
SAST findings
SCA findings
model metadata
Dockerfile
IaC
declared accelerator requirement
license information
artifact digest
```

### 13.3 Dynamic Evidence

What Kiste observes from actual execution or live systems.

Produced primarily by Inspect.

Examples:

```text
model successfully loads on an accelerator
runtime memory consumption
actual API endpoints
DAST findings
effective IAM behavior
live Kubernetes state
actual accelerator compatibility
latency
network behavior
runtime dependencies
```

### 13.4 Historical Evidence

What happened across lifecycle executions.

Produced primarily by Review, Deploy, and Monitor.

Examples:

```text
approved plan
rejected plan
deployment revision
rollback
runtime incidents
benchmark history
cost history
security history
model evaluation history
drift history
```

---

## 14. Model CV

The Model CV is not a replacement for Kiste's internal graphs.

It is a human- and machine-readable synthesis of accumulated evidence.

Conceptually:

```text
Model CV
├── identity
├── provenance
├── source revisions
├── declared intent
├── artifacts
├── dependencies
├── capabilities
├── static security evidence
├── dynamic security evidence
├── runtime compatibility
├── hardware compatibility
├── performance evidence
├── deployment history
├── policy history
├── evaluation history
├── cost history
├── incidents and drift
└── evidence references
```

A Model CV must distinguish:

```text
claimed
inferred
verified
observed
historical
```

It must not silently convert inference into fact.

Example:

```yaml
runtimeCompatibility:
  - target: nvidia-l40s
    status: verified
    evidenceRef: evidence://inspect/01J...
    observedAt: 2026-08-29T12:00:00Z

  - target: amd-mi300x
    status: inferred
    confidence: 0.72
    evidenceRef: evidence://read/01J...
```

The same evidence architecture may later support broader System CV or workload datasheet views without changing the underlying lifecycle contracts.

---

## 15. Model CV Is Evidence-Backed

Every important Model CV claim should be traceable.

Conceptually:

```text
CV claim
   ↓
evidence reference
   ↓
lifecycle execution
   ↓
Unit
   ↓
tool
   ↓
source/runtime observation
```

The CV should be able to answer questions such as:

```text
Which Git revision was tested?
Which dependencies were present?
Which scanner produced this finding?
Was accelerator compatibility inferred or actually tested?
Which plan deployed this revision?
Who approved it?
What runtime was used?
What performance was observed?
Has that evidence become stale?
Did later monitoring contradict the earlier result?
```

---

## 16. Tools and KisteUnits

Kiste does not absorb external tools into Core.

Canonical rule:

```text
Capability = what Kiste needs
Tool       = implementation surface
KisteUnit  = integration boundary
Adapter    = invocation mechanism
```

Example:

```text
security.sast
      ↓
Semgrep Unit
      ↓
local process / native / WASM
      ↓
Semgrep
```

Another example:

```text
runtime.kubernetes
      ↓
KubeVela integration Unit
      ↓
KubeVela
      ↓
Kubernetes
```

Tool replacement must not change capability meaning.

---

## 17. Standard Adapter Boundary

Kiste retains the four standard KisteUnit implementation mechanisms:

```text
native
local gRPC
remote gRPC
WASM
```

These are implementation details.

They are not lifecycle stages.

They are not four semantic KisteUnit kinds.

Normal users should express requirements such as:

```text
trusted
sandboxed
local-only
remote-denied
low-latency
```

rather than manually selecting transport mechanisms.

---

## 18. Kubernetes and KubeVela

Kubernetes is the default operating substrate for Kiste's initial deployment ecosystem.

Kiste does not attempt to hide that Kubernetes exists.

KubeVela/OAM is the default technical realization for Kubernetes application delivery where its model is appropriate.

Canonical relationship:

```text
Kiste
  intent
  evidence
  capabilities
  policy
  resolution
  planning
       ↓
KubeVela / OAM
       ↓
Kubernetes
```

Kiste must not rebuild functionality already mature in KubeVela, such as generic Kubernetes reconciliation or application-delivery primitives.

Kiste differentiation remains primarily before execution:

```text
understanding
evidence
capability requirement assembly
capability synthesis
policy
planning
trust
Model CV
```

---

## 19. Backend Boundary

Although Kubernetes/KubeVela is the default path, Plan and Deploy must preserve an implementation boundary.

Future implementations may include:

```text
direct Kubernetes
Argo CD
Flux
SkyPilot
Terraform/OpenTofu
cloud APIs
Slurm/HPC
edge runtimes
robotics/Omniverse infrastructure
other execution systems
```

The backend is selected because it satisfies capabilities.

The backend must not define Kiste's semantic architecture.

---

## 20. Policy Is Cross-Cutting

Policy is not a seventh Control Unit.

Policy constrains every Control Unit.

Examples:

```text
Read:
    which sources may be accessed

Inspect:
    which live systems may be observed

Plan:
    which implementations are allowed

Review:
    what approval is required

Deploy:
    what may mutate

Monitor:
    what drift or risk triggers action
```

Policy always overrides preference.

Policy enforcement mechanisms may themselves be provided by KisteUnits and existing policy engines.

---

## 21. Git and Evidence

Git remains Kiste's primary source history and declared-state audit trail.

Each lifecycle run should bind to:

```text
repository identity
requested ref
resolved commit
tree digest
Unit specification digest
intent digest
policy digest
static evidence digest
dynamic evidence digest
required capability graph digest
resolved capability graph digest
plan digest
approval digest
deployment result
monitoring result
```

Git alone is not sufficient for runtime evidence.

Kiste therefore links Git history with immutable evidence records.

---

## 22. State Rule

Kiste must distinguish:

```text
desired state
observed state
historical state
```

Observed state must not silently redefine user intent.

Canonical rule:

```text
desired = intent

starting point = observed state

plan = transition from observed toward desired
```

Existing infrastructure can reduce the required change.

It cannot silently change the intended outcome.

---

## 23. Safety Rules

The following rules are normative:

```text
Read and Inspect are non-mutating.

Plan is non-mutating.

Review does not execute the plan.

Deploy requires an approved immutable plan.

Unknown required capabilities block mutation.

Missing required evidence blocks mutation.

Stale dynamic evidence must be identified.

Changed source or policy may invalidate a plan.

Changed observed state may invalidate a plan.

Policy overrides preference.

Hard constraints cannot be compensated by scores.

Every inference records evidence and confidence.

Every deployment records execution evidence.

Monitor may cause the lifecycle to re-enter Read or Inspect.
```

Kiste fails closed when authorization, state, evidence, or capability resolution cannot be proven.

---

## 24. Suggested Artifact Layout

```text
.kiste/
  read/
    source-evidence.json
    static-evidence.json
    dependency-evidence.json
    security-static-evidence.json

  inspect/
    dynamic-evidence.json
    runtime-evidence.json
    security-dynamic-evidence.json
    inspection-result.json

  capabilities/
    declared-capabilities.json
    discovered-capabilities.json
    required-capability-graph.json
    candidate-implementation-graph.json
    resolved-capability-graph.json
    missing-capability-report.json
    rejected-implementation-report.json

  plans/
    latest.json
    execution-plan.json
    rollback-plan.json
    permission-plan.json

  reviews/
    latest.json
    latest.approved.json
    resolution-evidence.json
    policy-evidence.json

  deploy/
    execution-result.json

  monitor/
    health.json
    drift.json
    cost.json
    security.json

  evidence/
    index.json

  cv/
    model-cv.json
    model-cv.yaml
```

Exact filenames are implementation details unless separately standardized.

The semantic boundaries are normative.

---

## 25. Non-Goals

Phase 9.13A does not require:

```text
rewriting Kiste Core
splitting the six Control Units into six services
implementing every security scanner
building a new Kubernetes reconciler
forking KubeVela
replacing Git
standardizing TOSCA integration
standardizing Puccini integration
implementing a marketplace
implementing automatic cloud mutation
making all Model CV fields mandatory
building a universal optimizer
```

This phase standardizes architecture first.

---

## 26. Implementation Migration Principle

The implementation must follow the specification, not the reverse.

Migration should occur in this order:

```text
9.13A specification
      ↓
schema impact review
      ↓
implementation migration plan
      ↓
tests
      ↓
kiste-py implementation
      ↓
dogfood Kiste against itself
```

Existing compatibility code may remain temporarily where needed.

It must not redefine the new architecture.

---

## 27. Acceptance Criteria

Phase 9.13A is accepted only if:

```text
1. Kiste has exactly six canonical Control Units:
   Read, Inspect, Plan, Review, Deploy, Monitor.

2. Recommend, Apply, and Recheck are mapped to
   Review, Deploy, and Monitor rather than retained
   as competing lifecycle names.

3. Read owns source resolution and static evidence.

4. SAST, SCA, SBOM, secret scanning, and static IaC
   analysis can be represented as Read capabilities.

5. Inspect owns dynamic/runtime observation.

6. DAST, runtime probes, live infrastructure observation,
   hardware observation, and runtime security can be
   represented as Inspect capabilities.

7. Inspect produces RequiredCapabilityGraph.

8. Every inferred capability requirement can record
   evidence, source, and confidence.

9. Capability synthesis is explicitly a responsibility
   of the Plan Control Unit.

10. There is no seventh Capability Synthesis lifecycle stage.

11. Plan produces ResolvedCapabilityGraph and execution plans.

12. Policy overrides preference and applies across all
    Control Units.

13. Deploy requires an approved immutable plan.

14. Monitor records historical evidence and may trigger
    a new Read or Inspect cycle.

15. KisteUnit remains the common integration abstraction.

16. Native, local gRPC, remote gRPC, and WASM remain
    implementation mechanisms rather than lifecycle stages.

17. Kubernetes is recognized as the default operating
    substrate for the initial ecosystem.

18. KubeVela/OAM may serve as the default Kubernetes
    delivery implementation without replacing the
    Kiste object model.

19. Backend boundaries remain available for non-Kubernetes
    execution systems.

20. Model CV distinguishes declared, static/inferred,
    dynamic/observed, and historical evidence.

21. Important Model CV claims are traceable to evidence.

22. Observed state cannot silently override intent.

23. Unknown, stale, conflicting, or unauthorized state
    fails closed for mutation.

24. TOSCA/Puccini are explicitly deferred compatibility
    concerns rather than 9.13A dependencies.

25. Existing implementation changes occur only after this
    architecture specification is accepted.
```

---

## 28. Final Architecture Rule

```text
Read establishes what can be known without execution.

Inspect establishes what reality currently shows
and what capabilities are required.

Plan determines what should provide those capabilities
and what should change.

Review determines whether the plan is justified
and authorized.

Deploy executes only the approved immutable plan.

Monitor determines whether reality continues to match
the expected result.

KisteUnits provide the implementations.

Capabilities provide the vocabulary.

Policy provides the boundaries.

Git provides source history.

Evidence provides trust.

Model CV provides the accumulated, explainable record.
```

---

## 29. Phase 9.13A Summary

```text
                     Intent
                       │
                       ▼
               READ CONTROL UNIT
          source resolution + static analysis
                       │
                 StaticEvidence
                       │
                       ▼
             INSPECT CONTROL UNIT
        dynamic observation + requirement assembly
                       │
              RequiredCapabilityGraph
                       │
                       ▼
               PLAN CONTROL UNIT
       synthesis + resolution + change planning
                       │
       ResolvedCapabilityGraph + ExecutionPlan
                       │
                       ▼
              REVIEW CONTROL UNIT
            policy + risk + authorization
                       │
                  ApprovedPlan
                       │
                       ▼
              DEPLOY CONTROL UNIT
                    execution
                       │
                ExecutionEvidence
                       │
                       ▼
              MONITOR CONTROL UNIT
           health + drift + historical evidence
                       │
                       ├───────────────┐
                       ▼               │
                    Model CV           │
                                       │
                         drift/change ─┘
```

Canonical rule:

> Kiste converts intent and evidence into capability requirements, capability requirements into an explainable plan, and an approved plan into auditable observed history.
