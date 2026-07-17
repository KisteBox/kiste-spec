# Kiste v0.9.12 — Inspection-First Capability Assembly and Hook Adapter Standard

Status: Normative architecture clarification  
Release: `0.9.12`  
Theme: Assemble the required capability graph during inspection, before capability synthesis, using KisteUnits implemented through four standard hook adapters.

---

## 1. Core Ordering Rule

Capability requirement assembly is part of the **Inspect** phase.

It must happen before capability synthesis, implementation matching, preference scoring, graph optimization, and planning.

```text
Read
  ↓
Inspect
  ├── inspect intent
  ├── inspect workspace and Unit declarations
  ├── inspect codebase and repository graph
  ├── inspect current cloud/runtime state
  ├── inspect policy and environment constraints
  ├── produce evidence-backed facts
  └── assemble required capability graph
  ↓
Capability Synthesis
  ├── discover candidate KisteUnits
  ├── check capability compatibility
  ├── check trust, policy, and environment compatibility
  ├── apply hard constraints
  ├── score preferences and fit
  ├── optimize the implementation graph
  └── produce resolved capability graph
  ↓
Plan
  ├── calculate changes
  ├── calculate permissions
  ├── calculate operation order
  └── calculate rollback or containment
  ↓
Recommend → Apply → Recheck
```

Normative rule:

```text
Inspection determines what capabilities are required.
Synthesis determines which KisteUnits should provide them.
Planning determines what actions the resolved KisteUnits should perform.
```

These responsibilities must not be collapsed into one opaque optimizer step.

---

## 2. Lifecycle Boundary

### Read

`Read` resolves stable inputs for inspection:

```text
intent request
workspace definition
KisteUnit definitions
source locations
immutable Git revisions
available state-reading permissions
policy references
known environment references
```

Read does not synthesize an implementation stack.

### Inspect

`Inspect` gathers and normalizes evidence:

```text
codebase facts
repository facts
Unit facts
capability declarations
current cloud facts
current runtime facts
security and identity facts
hardware facts
policy constraints
drift and health facts
```

Inspect then assembles:

```text
DeclaredCapabilitySet
DiscoveredCapabilitySet
RequiredCapabilityGraph
MissingEvidenceReport
CapabilityConflictReport
```

### Capability Synthesis

Capability synthesis consumes the completed inspection result.

Inputs:

```text
RequiredCapabilityGraph
UnitCatalog
ToolCatalog
WorkspacePolicy
TrustConstraints
EnvironmentConstraints
CapabilityPreferences
CurrentStateDigest
```

Outputs:

```text
CandidateImplementationGraph
RejectedImplementationReport
PreferenceFitReport
ResolvedCapabilityGraph
ResolutionEvidence
```

### Plan

Plan consumes the resolved capability graph.

A plan must not be generated from an incomplete or stale required capability graph.

---

## 3. Inspect Result Contract

```yaml
apiVersion: kiste.dev/v0.9.12
kind: UnitInspectionResult

metadata:
  unitRef: payments/payment-api
  executionId: inspect-01J...

spec:
  intentDigest: sha256:...
  unitGraphDigest: sha256:...
  sourceRevision: 842ef607...
  policyDigest: sha256:...

status:
  facts:
    codebase: []
    infrastructure: []
    runtime: []
    security: []
    hardware: []

  capabilities:
    declared: []
    discovered: []

    requiredGraph:
      nodes: []
      edges: []

  evidence: []
  missingEvidence: []
  conflicts: []

  synthesisReady: true
```

`synthesisReady` must be `false` when:

```text
required evidence is missing
current state is too stale
required capability identity is unknown
capability dependencies are cyclic without an allowed cycle contract
policy cannot be evaluated
Unit graph cannot be resolved
```

Mutation-oriented synthesis must not proceed when `synthesisReady` is false.

---

## 4. Requirement Assembly Formula

Conceptually:

```text
RequiredCapabilityGraph = assemble(
  intent,
  workspace,
  Unit declarations,
  codebase facts,
  current cloud/runtime state,
  security and hardware facts,
  policy constraints,
  environment constraints
)
```

Capability synthesis is a separate function:

```text
ResolvedCapabilityGraph = synthesize(
  RequiredCapabilityGraph,
  available KisteUnits,
  available tools,
  policy,
  trust constraints,
  preferences,
  fit scores
)
```

The separation is important:

```text
Requirement assembly answers: What is needed?
Capability synthesis answers: What should provide it?
Plan generation answers: What should change?
```

---

## 5. KisteUnit Contribution During Inspect

KisteUnit is the atomic inspection and lifecycle object.

Each relevant Unit may contribute:

```text
facts
findings
provided capability declarations
required capability declarations
inferred capability requirements
current-state observations
confidence scores
evidence references
conflicts
missing evidence
```

Conceptual SDK contract:

```go
type InspectContributor interface {
    Inspect(ctx InspectContext) (InspectContribution, error)
}

type InspectContribution struct {
    Facts                []Fact
    Findings             []Finding
    DeclaredCapabilities []Capability
    DiscoveredCapabilities []Capability
    RequiredCapabilities []CapabilityRequirement
    Conflicts            []CapabilityConflict
    MissingEvidence      []EvidenceRequirement
    Evidence             []Evidence
}
```

The Control Unit merges Unit contributions into the global required capability graph.

---

## 6. Four Standard Hook Adapter Types

Kiste v0.9.12 supports four implementation adapters for KisteUnit lifecycle behavior:

```text
1. native
2. local gRPC
3. client/remote gRPC
4. WASM
```

These are implementation mechanisms.

They are not the public KisteUnit abstraction and should not normally appear in user-authored `kiste.yaml`.

```text
KisteUnit = public lifecycle and capability contract
Hook adapter = internal mechanism used to execute the Unit implementation
```

The selected adapter belongs in:

```text
KisteUnit package metadata
Core runtime configuration
resolved Unit lock data
execution evidence
```

---

## 7. Native Adapter

A native implementation executes inside the Kiste process or through a directly loaded trusted library.

Best suited for:

```text
bundled standard Units
Core Git and filesystem inspection
small trusted Go integrations
high-frequency local operations
```

Properties:

```text
highest performance
lowest isolation
shared process failure domain
trusted code only
```

Native inspection must still obey read-only Inspect permissions.

---

## 8. Local gRPC Adapter

A local gRPC implementation runs in a separate process on the same host.

Best suited for:

```text
Python integrations
language-independent Unit implementations
dependency-heavy adapters
local tools needing process isolation
```

Transport may use:

```text
Unix domain socket
localhost TCP
named pipe where supported
```

Core responsibilities:

```text
process startup
health checking
protocol negotiation
timeout enforcement
execution lease
shutdown and restart
circuit breaking
```

---

## 9. Client/Remote gRPC Adapter

Kiste Core acts as a client of a separately deployed KisteUnit service.

Best suited for:

```text
shared organization integrations
remote execution workers
central policy or identity services
large cloud, ML, and data integrations
cross-workspace managed services
```

Mandatory controls:

```text
authenticated service identity
mTLS or equivalent transport protection
protocol version negotiation
request timeout
idempotency key
execution lease
capability advertisement
immutable request and plan digests
```

Remote availability failure during Inspect may produce degraded or incomplete evidence.
It must not be interpreted as permission to infer that current state is safe.

---

## 10. WASM Adapter

A WASM implementation executes in a Kiste-managed sandbox.

Best suited for:

```text
portable inspection rules
third-party validators
policy checks
small transformations
semi-trusted extensions
deterministic analysis
```

Host access is capability-gated:

```text
filesystem read
network access
clock access
environment access
process access
secret-reference access
report output
```

WASM receives no host capability unless explicitly granted.

---

## 11. Common Hook Contract

All four adapters must expose the same normalized lifecycle contract.

```text
Read
Inspect
Plan
Recommend
Apply
Recheck
```

For the inspection-first requirement assembly flow, all adapters must normalize Inspect output to:

```text
Facts
Findings
Capability declarations
Capability requirements
Conflicts
Missing evidence
Confidence
Evidence
Observed state
```

Adapter choice must not alter capability semantics.

```text
The same Unit implemented through native, local gRPC, remote gRPC, or WASM must remain logically the same KisteUnit.
```

---

## 12. Workspace YAML Rule

Normal workspace YAML selects:

```text
intent
KisteUnits
required capabilities
policy
preferences
constraints
```

Normal workspace YAML does not select:

```text
native hook
local gRPC hook
remote gRPC hook
WASM hook
```

A workspace may express operational policy such as:

```yaml
spec:
  policies:
    unitExecution:
      untrustedCode:
        requireIsolation: sandboxed

      remoteExecution:
        effect: deny
```

Kiste resolves the adapter that satisfies those properties.

It should not require users to hardcode a hook transport.

---

## 13. Package Metadata and Lock Evidence

Example Unit package implementation metadata:

```yaml
apiVersion: kiste.dev/v0.9.12
kind: UnitPackage

metadata:
  name: kubeflow-unit
  version: 0.1.0

spec:
  unitRef: ml/kubeflow

  implementation:
    adapter: local-grpc
    protocol: kiste.unit/v1alpha1
    executable: kiste-unit-kubeflow
```

Resolved evidence:

```yaml
status:
  resolvedUnits:
    ml/kubeflow:
      package: kubeflow-unit@0.1.0
      packageDigest: sha256:...

      implementation:
        adapter: local-grpc
        protocol: kiste.unit/v1alpha1
        executableDigest: sha256:...
```

This makes execution reproducible without exposing hook details as the normal workspace model.

---

## 14. KisteUnit SDK Requirements

The SDK must provide:

```text
shared lifecycle contracts
shared InspectContribution model
native adapter SDK
local gRPC server SDK
remote gRPC client/server protocol
WASM guest SDK
WASM host contract
adapter conformance tests
inspection read-only tests
capability evidence tests
serialization compatibility tests
version negotiation tests
failure and timeout tests
```

Every adapter implementation must pass the same Unit contract test suite.

Required conformance rule:

```text
Given the same Unit inputs and observed state,
all conforming adapters must produce semantically equivalent normalized results.
```

---

## 15. Role of the Control Unit

The Kiste Control Unit coordinates the full sequence:

```text
resolve intent
resolve Unit graph
run Read
run Inspect contributors through supported adapters
merge facts and evidence
assemble required capability graph
validate synthesis readiness
invoke capability synthesis
record selected KisteUnits and adapters
invoke Plan on the resolved Unit graph
```

The Control Unit does not allow the capability synthesizer to inspect arbitrary mutable state directly.

The synthesizer consumes the evidence-backed Inspect result.

This preserves reproducibility and prevents hidden state discovery inside optimization.

---

## 16. Generated Outputs

```text
.kiste/inspect/resolved-intent.json
.kiste/inspect/codebase-facts.json
.kiste/inspect/observed-cloud-state.json
.kiste/inspect/observed-runtime-state.json
.kiste/inspect/unit-contributions.json
.kiste/inspect/required-capability-graph.json
.kiste/inspect/missing-evidence-report.json
.kiste/inspect/capability-conflict-report.json
.kiste/inspect/inspection-result.json

.kiste/synthesis/candidate-implementation-graph.json
.kiste/synthesis/rejected-implementation-report.json
.kiste/synthesis/preference-fit-report.json
.kiste/synthesis/resolved-capability-graph.json
.kiste/synthesis/resolution-evidence.json

.kiste/lock/resolved-units.yaml
```

The directory separation is deliberate:

```text
inspect/ contains evidence and requirements.
synthesis/ contains implementation selection.
plan/ contains proposed actions.
```

---

## 17. Safety Rules

```text
Inspect is read-only.
Requirement contributors cannot mutate external state.
Hook adapter type does not change lifecycle permissions.
Missing observations are recorded as missing evidence.
Stale current state is explicitly marked.
Capability synthesis cannot silently discover mutable state.
Synthesis cannot weaken required capabilities.
Policy overrides preference and optimization.
Plan is invalid when bound inspection evidence becomes stale.
Apply requires an approved plan bound to the resolved capability graph.
```

---

## 18. Acceptance Criteria

v0.9.12 satisfies this clarification only if:

```text
1. Capability requirement assembly is explicitly part of Inspect.
2. Requirement assembly completes before capability synthesis.
3. Capability synthesis completes before Plan.
4. Inspect produces an evidence-backed required capability graph.
5. The synthesizer consumes Inspect output instead of performing hidden state discovery.
6. KisteUnit is the atomic contributor to inspection and requirement assembly.
7. Native adapter is supported.
8. Local gRPC adapter is supported.
9. Client/remote gRPC adapter is supported.
10. WASM adapter is supported.
11. All adapters use the same lifecycle and result contracts.
12. Hooks remain implementation details rather than normal workspace YAML fields.
13. The SDK provides adapter builders and conformance tests.
14. Core records the selected implementation adapter in lock and evidence outputs.
15. Inspection, synthesis, and planning outputs are separated.
```

---

## 19. Final Rule

```text
Kiste v0.9.12 follows this order:

Read
  -> Inspect intent, codebase, Unit declarations, policy, and current state
  -> assemble the required capability graph
  -> validate synthesis readiness
  -> synthesize the best-fit allowed KisteUnit graph
  -> Plan
  -> Recommend
  -> Apply
  -> Recheck

The four hook adapters are:

  native
  local gRPC
  client/remote gRPC
  WASM

They implement KisteUnits.
They do not replace KisteUnits.
They do not normally appear in workspace YAML.
```
