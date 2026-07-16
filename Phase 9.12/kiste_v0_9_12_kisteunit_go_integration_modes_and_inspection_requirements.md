# Kiste v0.9.12 — KisteUnit Go Integration Modes and Inspect-Phase Capability Requirements

Status: Normative architecture clarification  
Release: `0.9.12`  
Theme: KisteUnits inspect the system and assemble the required capability graph; KisteUnit implementations integrate Go-native tools through four standard execution and communication modes.

---

## 1. Two Separate Concepts

v0.9.12 must keep these concepts separate:

```text
A. Capability requirement assembly
B. KisteUnit implementation mode
```

Capability requirement assembly belongs to the `Inspect` phase.

The four implementation modes describe how a KisteUnit implementation communicates with or embeds a tool, especially a Go-native tool.

They do not define four different semantic kinds of KisteUnit.

```text
KisteUnit identity and contract remain the same.
Only the implementation and communication boundary changes.
```

---

## 2. Normative Lifecycle Order

```text
Read
  ↓
Inspect
  ├── resolve user intent
  ├── inspect workspace and KisteUnit declarations
  ├── inspect codebase and repository graph
  ├── inspect current cloud state
  ├── inspect current runtime state
  ├── inspect security, hardware, and policy state
  ├── collect evidence-backed facts
  ├── infer required capabilities
  ├── expand capability dependencies
  └── produce RequiredCapabilityGraph
  ↓
Capability Synthesis
  ├── discover candidate KisteUnits
  ├── match implementations to required capabilities
  ├── reject incompatible implementations
  ├── apply policy and trust constraints
  ├── score preference and fit
  ├── optimize the implementation graph
  └── produce ResolvedCapabilityGraph
  ↓
Plan
  ↓
Recommend
  ↓
Apply
  ↓
Recheck
```

Normative rule:

```text
Inspect answers: What capabilities are required?
Synthesis answers: Which KisteUnits should provide them?
Plan answers: What changes should those resolved Units perform?
```

Capability synthesis must not secretly perform its own mutable-state inspection.

---

## 3. Capability Requirement Assembly Is an Inspect Responsibility

The `Inspect` phase consumes:

```text
intent
workspace definition
KisteUnit declarations
codebase evidence
Git and repository state
dependency manifests
infrastructure definitions
current cloud state
current runtime state
current security and identity state
hardware state
policy constraints
environment constraints
```

The `Inspect` phase produces:

```text
DeclaredCapabilitySet
DiscoveredCapabilitySet
RequiredCapabilityGraph
MissingEvidenceReport
CapabilityConflictReport
InspectionEvidenceBundle
SynthesisReadinessDecision
```

The required graph is not a list manually maintained by the user.

It is an evidence-backed result of inspection.

Conceptually:

```text
RequiredCapabilityGraph = inspectAndAssemble(
  intent,
  workspace,
  Unit declarations,
  codebase,
  current state,
  policy,
  environment
)
```

---

## 4. Inspect Output Contract

```yaml
apiVersion: kiste.dev/v0.9.12
kind: InspectionResult

metadata:
  unitRef: payments/payment-api
  executionId: inspect-01J...

spec:
  intentDigest: sha256:...
  workspaceDigest: sha256:...
  unitGraphDigest: sha256:...
  sourceRevision: 842ef607...
  observedStateDigest: sha256:...
  policyDigest: sha256:...

status:
  facts:
    codebase: []
    repository: []
    cloud: []
    runtime: []
    security: []
    hardware: []

  capabilities:
    declared: []
    discovered: []

    requiredGraph:
      nodes: []
      edges: []

  conflicts: []
  missingEvidence: []
  evidence: []

  synthesisReady: true
```

`synthesisReady` must be false when:

```text
required observations are missing
observed state is too stale
policy cannot be evaluated
required capability identity is unknown
capability dependencies cannot be resolved
Unit graph resolution failed
```

---

## 5. KisteUnit Is the Inspect Contributor

KisteUnit is the atomic inspection and lifecycle object.

Each Unit may contribute:

```text
facts
findings
current-state observations
provided capability declarations
required capability declarations
inferred capability requirements
capability conflicts
missing evidence
confidence scores
evidence references
```

Conceptual SDK contract:

```go
type InspectContributor interface {
    Inspect(ctx InspectContext) (InspectContribution, error)
}

type InspectContribution struct {
    Facts                  []Fact
    Findings               []Finding
    Observations           []Observation
    DeclaredCapabilities   []Capability
    DiscoveredCapabilities []Capability
    RequiredCapabilities   []CapabilityRequirement
    Conflicts              []CapabilityConflict
    MissingEvidence        []EvidenceRequirement
    Evidence               []Evidence
}
```

The Kiste Control Unit merges contributions into the global `RequiredCapabilityGraph`.

---

## 6. Four KisteUnit Implementation and Communication Modes

A KisteUnit implementation can integrate a Go-native tool through four standard modes:

```text
1. Native Go
2. Local gRPC process
3. Client/remote gRPC service
4. WASM
```

These modes answer:

```text
How does this KisteUnit implementation communicate with, embed, or execute the tool integration?
```

They do not answer:

```text
What capabilities does the Unit provide?
What does the Unit require?
What lifecycle does the Unit follow?
```

Those remain part of the common KisteUnit contract.

---

## 7. Native Go Mode

The KisteUnit implementation directly uses Go libraries or is compiled into the Kiste runtime.

```text
Kiste Core process
  -> native KisteUnit implementation
  -> Go library/API
  -> Go-native tool or object model
```

Best suited for:

```text
trusted bundled standard KisteUnits
Kubernetes client-go integrations
Docker/Moby/containerd libraries
OCI libraries
Helm or Kustomize libraries
high-performance local inspection
```

Properties:

```text
highest performance
lowest serialization overhead
direct access to Go types
shared process failure domain
lowest isolation
trusted code only
```

Native Go is the preferred mode when Kiste and the integrated tool already share a stable Go API and the implementation is trusted.

---

## 8. Local gRPC Process Mode

The KisteUnit implementation runs as a separate local process and communicates with Kiste Core through gRPC.

```text
Kiste Core
  -> local gRPC
  -> local KisteUnit process
  -> Go-native tool, Go CLI, API, or SDK
```

The local process may itself be written in:

```text
Go
Python
Rust
Java
another gRPC-capable language
```

Best suited for:

```text
dependency-heavy integrations
process isolation
separate release cycles
language-independent Units
wrapping an existing Go CLI or daemon
```

Supported local transports:

```text
Unix domain socket
localhost TCP
named pipe where supported
```

Core manages:

```text
process startup
health check
protocol negotiation
timeout
execution lease
restart
shutdown
circuit breaker
```

---

## 9. Client/Remote gRPC Mode

Kiste Core acts as a client of a separately deployed KisteUnit service.

```text
Kiste Core
  -> authenticated remote gRPC
  -> remote KisteUnit service
  -> Go-native platform or tool
```

Best suited for:

```text
shared platform integrations
remote execution workers
central Kubernetes or cloud services
organization IAM and policy services
large Kubeflow or data-platform integrations
cross-workspace managed services
```

Mandatory controls:

```text
mTLS or equivalent workload identity
protocol version negotiation
capability advertisement
request timeout
idempotency key
execution lease
immutable request digest
immutable approved plan digest for mutation
```

A remote failure during Inspect produces missing or stale evidence.
It must never be treated as proof that the current state is safe.

---

## 10. WASM Mode

The KisteUnit implementation runs in a Kiste-controlled WebAssembly sandbox.

```text
Kiste Core
  -> WASM host contract
  -> sandboxed KisteUnit module
  -> explicitly granted host functions
```

WASM is useful for:

```text
portable inspectors
policy and validation rules
third-party extensions
small deterministic transformations
restricted integration logic
semi-trusted plugins
```

WASM does not directly receive unrestricted access to Go tools.
Instead, Kiste exposes narrow host capabilities that may internally call Go-native libraries.

Example host capabilities:

```text
workspace.read
repository.read
kubernetes.object.read
oci.manifest.read
report.write
```

Unlisted host capabilities are denied.

---

## 11. Common KisteUnit Contract Across All Four Modes

All four modes must implement the same normalized KisteUnit lifecycle:

```text
Read
Inspect
Plan
Recommend
Apply
Recheck
```

For `Inspect`, every mode must return the same semantic result model:

```text
Facts
Findings
Observations
Declared capabilities
Discovered capabilities
Required capability contributions
Conflicts
Missing evidence
Confidence
Evidence
```

Rule:

```text
Changing Native Go to local gRPC, remote gRPC, or WASM must not change the logical identity of the KisteUnit.
```

The implementation mode may affect:

```text
performance
isolation
trust level
latency
deployment model
failure boundary
```

It must not change capability meaning.

---

## 12. Implementation Modes Are Not Normal Workspace YAML Fields

User-authored `kiste.yaml` should normally declare:

```text
intent
KisteUnits
capability requirements
policy
preferences
trust and isolation constraints
```

It should not normally declare:

```text
native-go
local-grpc
remote-grpc
wasm
```

The implementation mode belongs in:

```text
KisteUnit package metadata
Core runtime registry
resolved Unit lock file
execution evidence
```

Workspace policy may express properties:

```yaml
spec:
  policies:
    unitExecution:
      untrustedUnits:
        requireIsolation: sandboxed

      remoteExecution:
        effect: deny
```

Kiste then resolves a compatible implementation mode.

---

## 13. KisteUnit Package Metadata

```yaml
apiVersion: kiste.dev/v0.9.12
kind: UnitPackage

metadata:
  name: kubernetes-inspector
  version: 0.1.0

spec:
  unitRef: std/kubernetes-inspector

  implementation:
    mode: native-go
    module: github.com/KisteBox/kiste-core/standard-units/kubernetes
```

Local gRPC example:

```yaml
spec:
  implementation:
    mode: local-grpc
    protocol: kiste.unit/v1alpha1
    executable: kiste-unit-kubeflow
```

Remote gRPC example:

```yaml
spec:
  implementation:
    mode: remote-grpc
    protocol: kiste.unit/v1alpha1
    serviceRef: platform/kubeflow-unit
```

WASM example:

```yaml
spec:
  implementation:
    mode: wasm
    module: kubernetes-policy.wasm
    digest: sha256:...
```

---

## 14. Capability Synthesis Boundary

Capability synthesis begins only after `Inspect` produces a valid required graph.

```text
Inspect output:
  RequiredCapabilityGraph

Synthesis input:
  RequiredCapabilityGraph
  available KisteUnits
  available implementation modes
  available tools
  policy
  trust constraints
  preferences
  fit scores
```

Synthesis output:

```text
ResolvedCapabilityGraph
ResolvedUnitGraph
SelectedImplementationModes
RejectedCandidateReport
PreferenceFitReport
ResolutionEvidence
```

The synthesizer may select among multiple implementations of the same Unit or capability.

Example:

```text
runtime.kubernetes.inspect
  -> std/kubernetes-inspector via Native Go
  -> company/kubernetes-inspector via remote gRPC
  -> third-party/kubernetes-validator via WASM
```

Policy and fit determine the selected implementation.

---

## 15. KisteUnit SDK Requirements

The SDK must include:

```text
common KisteUnit lifecycle contract
InspectContribution contract
Native Go implementation SDK
local gRPC server SDK
remote gRPC client/server protocol
WASM guest SDK
WASM host interface
Go-tool adapter helpers
capability evidence builders
implementation conformance tests
read-only Inspect safety tests
serialization compatibility tests
protocol version tests
failure and timeout tests
```

All four implementation modes must pass the same KisteUnit contract tests.

---

## 16. Generated Outputs

Inspect outputs:

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
```

Synthesis outputs:

```text
.kiste/synthesis/candidate-unit-graph.json
.kiste/synthesis/rejected-candidates.json
.kiste/synthesis/preference-fit-report.json
.kiste/synthesis/resolved-capability-graph.json
.kiste/synthesis/resolved-unit-graph.json
.kiste/synthesis/selected-implementation-modes.json
.kiste/synthesis/resolution-evidence.json
```

Lock and evidence:

```text
.kiste/lock/resolved-units.yaml
.kiste/evidence/unit-implementation-evidence.json
```

---

## 17. Safety Rules

```text
Inspect is read-only.
Capability requirement assembly is completed during Inspect.
Implementation mode does not change lifecycle permission limits.
Missing tool or state access becomes missing evidence.
Stale state must be marked.
Capability synthesis cannot weaken required capabilities.
Capability synthesis cannot secretly inspect mutable state.
Policy overrides preference and optimization.
Plan binds to inspection and synthesis digests.
Apply requires an approved immutable plan.
```

---

## 18. Acceptance Criteria

v0.9.12 satisfies this standard only if:

```text
1. Capability requirement assembly is part of Inspect.
2. Inspect produces RequiredCapabilityGraph.
3. Capability synthesis starts only after Inspect completes.
4. Planning starts only after synthesis completes.
5. KisteUnit is the atomic contributor to inspection.
6. Native Go integration mode is supported.
7. Local gRPC process integration mode is supported.
8. Client/remote gRPC integration mode is supported.
9. WASM integration mode is supported.
10. The four modes integrate KisteUnits with Go-native tools and tool APIs.
11. The four modes are not four semantic KisteUnit kinds.
12. All modes implement the same KisteUnit contract.
13. Implementation mode does not normally appear in workspace YAML.
14. Selected mode is recorded in lock and evidence outputs.
15. SDK provides builders and conformance tests for all four modes.
```

---

## 19. Final Rule

```text
Capability requirement assembly belongs to Inspect.

Inspect produces the evidence-backed RequiredCapabilityGraph.

Capability synthesis consumes that graph and selects the best-fit allowed KisteUnits and implementations.

A KisteUnit may integrate a Go-native tool through:

  Native Go
  Local gRPC process
  Client/remote gRPC service
  WASM

These are implementation and communication modes of KisteUnit.
They are not separate semantic Unit kinds.
They do not normally appear in user-authored kiste.yaml.
```
