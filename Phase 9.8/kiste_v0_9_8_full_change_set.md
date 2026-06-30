# Kiste v0.9.8 — Full Change Set

Status: Draft canonical release spec  
Release: `0.9.8`  
Theme: SDK expansion, decentralized KisteUnits, Kubernetes-lized projects, smarter inspect, validation/review capabilities, self-test, and tool weaving

---

## 1. Release Theme

Kiste v0.9.8 expands Kiste from a foundation model into an SDK-driven platform model.

v0.9.8 does four important things:

```text
1. Removes plugin mode and makes KisteUnit the only extension object.
2. Expands the SDK so developers can build KisteUnits and weave existing tools into Kiste.
3. Kubernetes-lizes projects as the first real runtime path.
4. Adds hardware, security, AI, validation, review, self-test, and minimum operating capabilities.
```

One-sentence definition:

```text
Kiste v0.9.8 expands the SDK, removes plugin mode, moves KisteUnits to decentralized Git-style repositories, Kubernetes-lizes projects as the first real runtime path, adds hardware/security/AI capability analysis, and defines how existing tools are woven into Kiste through capabilities.
```

---

## 2. Remove Plugin Mode

Old model:

```text
plugin
intent
helper
provider
unit
```

New model:

```text
There are no plugins.
There are only KisteUnits.
```

A KisteUnit can be used as:

```text
top-level intent
dependency
provider
runtime unit
validator
review unit
self-test unit
SDK development unit
self-improvement unit
third-party extension
```

These are uses, not modes.

Do not use:

```yaml
mode: plugin
type: plugin
```

Use entrypoints and capabilities instead:

```yaml
apiVersion: kiste.dev/v0.9.8
kind: KisteUnit

metadata:
  name: kubernetes-runtime

spec:
  entrypoints:
    top_level:
      allowed: false
    dependency:
      allowed: true

  provides:
    capabilities:
      - runtime.kubernetes
      - runtime.health
      - runtime.drift
```

Final rule:

```text
KisteUnit behavior is derived from entrypoints, capabilities, policy, Git compatibility, lifecycle support, and trust policy.
```

---

## 3. Decentralized KisteUnit Repositories

KisteUnits move away from a central plugin registry.

They should work like Go modules: normal Git-style repositories with versioned module paths.

Examples:

```text
github.com/KisteBox/kiste-unit-kubernetes@v0.1.0
github.com/company/internal-cost-unit@v0.3.2
gitlab.com/team/kiste-unit-security@v1.2.0
internal.git.company.com/platform/kiste-unit-aws@v0.4.0
```

Workspace declaration:

```yaml
spec:
  units:
    requires:
      - module: github.com/KisteBox/kiste-unit-kubernetes
        version: v0.1.0

      - module: github.com/company/internal-ai-serving-unit
        version: v0.2.1
```

Unit resolution rule:

```text
KisteUnit resolution must not execute unit code.
```

The resolver should only fetch metadata, validate schema, validate capabilities, verify trust policy, and update the lock file.

---

## 4. Add `kiste.unit.lock`

v0.9.8 introduces:

```text
kiste.unit.lock
```

Purpose:

```text
pin unit versions
pin commits
record checksums
make unit resolution reproducible
support review for unit changes
```

Example:

```yaml
apiVersion: kiste.dev/v0.9.8
kind: KisteUnitLock

spec:
  units:
    - module: github.com/KisteBox/kiste-unit-kubernetes
      version: v0.1.0
      resolved: github.com/KisteBox/kiste-unit-kubernetes
      commit: 8f3ab21
      checksum: sha256:example
```

Rules:

```text
No production unit without pinning.
No version change without review.
No new unit without review.
No unit source outside allowed hosts.
No code execution during unit resolution.
```

---

## 5. Expand the SDK

v0.9.8 makes the SDK the main path for building KisteUnits and weaving existing tools into Kiste.

SDK areas:

```text
kiste
kiste.unit
kiste.capability
kiste.provider
kiste.validate
kiste.inspect
kiste.plan
kiste.review
kiste.tool
kiste.kubernetes
```

SDK responsibilities:

```text
load workspace
load KisteUnit
validate unit contract
declare capabilities
declare provider contracts
create PatchSet
create ActionPlan
create RuntimePlan
create ReviewReport
create InspectReport
create MonitorReport
wrap existing tools
normalize tool output into Kiste facts
```

Example lifecycle API:

```python
import kiste

workspace = kiste.Workspace.load(".")
inspection = kiste.inspect.run(workspace)

plan = kiste.plan.run(
    workspace,
    intent="kubernetes-lize-project",
)

review = kiste.review.run(plan)

if review.passed:
    kiste.deploy.run(review.approved_plan)
```

Example KisteUnit SDK:

```python
from kiste.unit import UnitBuilder

unit = (
    UnitBuilder("kubernetes-runtime")
    .provides("runtime.kubernetes")
    .provides("runtime.manifest_validation")
    .provides("runtime.health")
    .requires("git.update")
    .requires("secrets.refs")
    .policy(no_secret_values=True)
    .build()
)
```

Core rule:

```text
SDK is how tools and KisteUnits enter Kiste.
Core remains pure.
CLI remains thin.
```

---

## 6. Self-Improvement Becomes KisteUnit SDK / Workspace

Self-improvement is not special core logic.

It becomes the first example of a KisteUnit development workspace.

Meaning:

```text
The same SDK path is used for:
  Kiste improving itself
  developers building third-party KisteUnits
  teams building internal units
  vendors building commercial units
```

Self-improvement / unit SDK supports:

```text
unit scaffold
unit validation
unit test harness
sample workspace runner
mock WorkspaceContext
mock WorkspaceAccessPolicy
documentation generation
unit packaging
unit registry/index metadata later
```

Rule:

```text
Self-improvement must use Kiste like a normal advanced user.
It must not be imported by kiste_core.
It must not bypass GitUpdatePlan, review, or approval.
```

---

## 7. Kubernetes-Lize Projects

v0.9.8 makes Kubernetes the first real runtime path.

“Kubernetes-lize” means:

```text
Read an existing project and generate a safe Kubernetes-ready structure through Kiste plans.
```

It does not mean raw `kubectl`.

It does not mean direct cluster apply.

Kiste should generate something like:

```text
k8s/
  base/
    deployment.yaml
    service.yaml
    configmap.yaml
    secret-refs.yaml
  overlays/
    dev/
      kustomization.yaml
    prod/
      kustomization.yaml
```

Flow:

```text
kiste read
  detect app type, ports, Dockerfile, env vars, health endpoint

kiste inspect
  analyze runtime needs, secrets, hardware, security, resources

kiste plan
  generate Kubernetes PatchSet and RuntimePlan

kiste review
  validate manifests, SecretRef, resources, risk, rollback

kiste deploy
  commit manifests or apply approved plan

kiste monitor
  check rollout, pod health, events, drift
```

---

## 8. Kubernetes Is the First Real Provider

Kubernetes provides runtime capabilities.

Minimum Kubernetes capabilities:

```text
runtime.kubernetes
runtime.namespace
runtime.workload
runtime.service
runtime.config_ref
runtime.secret_ref
runtime.dry_run
runtime.manifest_validation
runtime.health
runtime.rollout_status
runtime.drift
runtime.events
```

Optional later:

```text
runtime.ingress
runtime.network_policy
runtime.rbac
runtime.resource_quota
runtime.autoscaling
runtime.storage_class
runtime.service_account
```

Boundary:

```text
Kubernetes is runtime.
Kubernetes is not cloud provisioning.
Kubernetes is not CI/CD.
Kubernetes is not the secret backend.
Kubernetes is not the model registry.
```

---

## 9. Add Hardware Analysis to Inspect

`kiste inspect` becomes smarter.

Hardware inspect capabilities:

```text
inspect.hardware
inspect.cpu
inspect.architecture
inspect.memory
inspect.disk
inspect.network
inspect.gpu
inspect.accelerator
inspect.storage_io
inspect.runtime_resource
inspect.node_constraint
inspect.cost_resource_signal
```

Kiste should detect:

```text
CPU architecture: x86_64, arm64
minimum memory
disk/storage needs
network ports
GPU requirement
container resource requests
container resource limits
stateful vs stateless workload
latency sensitivity
edge suitability
cloud suitability
Kubernetes node requirements
```

Example hardware report:

```json
{
  "schema": "kiste.hardware-inspection-report/v0.9.8",
  "status": "passed_with_warnings",
  "findings": [
    {
      "capability": "inspect.memory",
      "message": "No memory limit declared for backend container",
      "severity": "warning"
    },
    {
      "capability": "inspect.architecture",
      "message": "Docker image appears compatible with linux/amd64 and linux/arm64",
      "severity": "info"
    }
  ]
}
```

---

## 10. Add Security Analysis to Inspect

Security inspect capabilities:

```text
inspect.security
inspect.secret_leak
inspect.secret_ref
inspect.iam
inspect.rbac
inspect.network_exposure
inspect.container_security
inspect.dockerfile_security
inspect.dependency_risk
inspect.supply_chain
inspect.policy_boundary
inspect.privilege
inspect.runtime_security
inspect.data_boundary
inspect.auditability
```

Kiste should check:

```text
plaintext secrets
.env committed or planned
unsafe Dockerfile patterns
root container user
privileged container
public ingress exposure
wildcard RBAC
missing resource limits
unsafe IAM permissions
untrusted image source
missing SBOM reference
missing rollback plan
missing audit trail
```

Rule:

```text
Kiste inspect must block plans that expose secrets, bypass policy, or create unsafe runtime defaults.
```

Example security report:

```json
{
  "schema": "kiste.security-inspection-report/v0.9.8",
  "status": "failed",
  "blockers": [
    {
      "capability": "inspect.secret_leak",
      "message": "Plaintext secret detected in .env",
      "severity": "blocker"
    }
  ],
  "warnings": [
    {
      "capability": "inspect.container_security",
      "message": "Container runs as root",
      "severity": "warning"
    }
  ]
}
```

---

## 11. Add AI Capability Themes

v0.9.8 adds AI capability names, but not AI tool integrations yet.

Capability groups:

```text
ai.*
model.*
dataset.*
inference.*
agent.*
eval.*
embedding.*
vector.*
rag.*
gpu.*
mlops.*
safety.*
```

Examples:

```text
model.artifact
model.registry
model.deploy
model.rollback
model.eval

dataset.catalog
dataset.version
dataset.lineage
dataset.quality

inference.endpoint
inference.scaling
inference.latency_monitor
inference.cost_monitor

agent.runtime
agent.tool_registry
agent.audit

rag.pipeline
rag.retriever
rag.chunking
rag.citation
rag.eval

gpu.node_pool
gpu.scheduling
gpu.quota
gpu.cost

safety.content_policy
safety.data_boundary
safety.audit_log
```

Not integrated yet:

```text
Kaggle
Hugging Face runtime
MLflow
Kubeflow
Ray
LangChain
LlamaIndex
vector databases
model providers
```

---

## 12. SDK Tool-Weaving Model

This is one of the biggest v0.9.8 changes.

Existing tools should be woven into Kiste, not hardcoded into Kiste.

Tool weaving means the SDK can:

```text
1. declare what capabilities the tool provides
2. call the tool safely
3. normalize tool output into Kiste facts
4. convert tool changes into PatchSet or ActionPlan
5. validate before mutation
6. require review before execution
7. write evidence reports
8. monitor results
```

Tool output normalization:

```text
kubectl output        -> RuntimeFacts
Kubernetes manifest  -> RuntimePlan
Dockerfile scan      -> SecurityFinding
boto3 inventory      -> CloudFacts
Pulumi preview       -> ActionPlanEvidence
Ansible check        -> SelfTestReport
GitHub PR            -> ReviewEvidence
```

Tool weaving modes:

```text
observe-only
validate-only
render-only
render-and-apply
action-plan
hybrid
```

Example adapter interface:

```python
class KisteToolAdapter:
    def capabilities(self) -> list[str]:
        ...

    def read(self, context) -> "FactReport":
        ...

    def inspect(self, context) -> "InspectReport":
        ...

    def plan(self, context) -> "PlanFragment":
        ...

    def validate(self, target) -> "ValidationReport":
        ...

    def review(self, plan) -> "ReviewEvidence":
        ...

    def deploy(self, approved_plan) -> "ExecutionReport":
        ...

    def monitor(self, context) -> "MonitorReport":
        ...
```

Final rule:

```text
Existing tools must be woven into Kiste.
Kiste must not be bent around existing tools.
```

---

## 13. Expand `validate.*`

Validation becomes a capability layer.

It can be used by:

```text
kiste inspect
kiste review
kiste plan
Python SDK
KisteUnit SDK
self-improvement workspace
optional expert CLI: kiste validate
```

Validation capabilities:

```text
validate.schema
validate.workspace
validate.manager
validate.unit
validate.unit_lock
validate.capability
validate.provider_contract
validate.policy
validate.git_update_plan
validate.action_plan
validate.review_plan
validate.deployment_plan
validate.rollback_plan
validate.runtime_plan
validate.kubernetes_manifest
validate.kubernetes_dry_run
validate.ai_capability
validate.secret_ref
validate.security_boundary
validate.cost_boundary
validate.architecture_boundary
validate.sdk_contract
validate.third_party_unit
```

Rule:

```text
Validation is non-mutating by default.
Mutating validation becomes a reviewed SelfTestPlan.
```

---

## 14. Expand `review.*`

`kiste review` becomes capability-bearing.

It can run:

```text
review.plan_validate
review.policy_check
review.diff_summary
review.risk_score
review.security_check
review.secret_check
review.cost_check
review.dependency_check
review.rollback_check
review.approval_record
review.audit_log
```

It can also call:

```text
validate.*
selftest.*
readiness.*
audit.*
```

Review answers:

```text
Is this plan valid?
Is it safe?
Is it reversible?
Is it testable?
Is it ready to deploy?
```

---

## 15. Add Self-Test and Stress-Test Concepts

Self-test:

```text
Can this deployment or plan prove it is safe before real deploy?
```

Stress test:

```text
A high-intensity self-test that pushes load, scale, failure, or cost boundary.
```

Capabilities:

```text
selftest.schema
selftest.unit_resolution
selftest.capability_resolution
selftest.policy_gate
selftest.git_update_plan
selftest.action_plan
selftest.kubernetes_dry_run

stress.load
stress.scale
stress.quota
stress.failure_recovery

loadtest.http
loadtest.inference_endpoint

chaos.pod_restart
chaos.node_pressure

resilience.rollback
resilience.recovery_probe

performance.latency
performance.throughput
performance.cost_under_load
```

Rule:

```text
If the test only checks, it is validation.
If the test runs the system, it is self-test.
If the test pushes the system hard, it is stress test.
```

---

## 16. Minimum Kiste Operating Capability Kernel

Kiste must run without external tools.

The Minimum Operating Capability Kernel is the smallest set of capabilities required for Kiste to operate.

Minimum capabilities:

```text
workspace.read
workspace.status_read

repo.discover
repo.read_metadata

git.state_read

schema.validate
validate.workspace
validate.capability
validate.policy
validate.plan
validate.secret_ref

capability.registry_read
capability.resolve

policy.read
policy.validate

graph.workspace_build
graph.capability_build
graph.policy_build
graph.action_build

plan.generate
plan.write

review.validate
review.blocker_report
review.warning_report
review.decision_record

audit.write
report.write
output.write

monitor.local
```

This lets Kiste do:

```text
read
inspect
validate
plan
review
write reports
monitor local state
```

without Kubernetes, GitHub, boto3, cloud, or third-party units.

Final rule:

```text
Kiste core must be able to operate in read-only planning mode with only the Minimum Operating Capability Kernel.
```

---

## 17. Optional Capability Tiers

### Tier 0 — Core Read-Only Kiste

```text
read
inspect
plan
review
monitor local
```

### Tier 1 — Local Git Mutation

```text
git.patchset
git.branch_create
git.commit_create
approval.record
```

### Tier 2 — Remote Git / PR

```text
git.push_branch
git.pull_request_create
git.status_read
```

### Tier 3 — Kubernetes Runtime

```text
runtime.kubernetes
runtime.manifest_generate
runtime.manifest_validation
runtime.dry_run
runtime.health
runtime.drift
```

### Tier 4 — v1 Easy Mode

```text
git.review
cicd.runner
cloud.inventory
cloud.action_plan
cloud.action_execute_approved
monitor.job_status
```

Mapped to:

```text
GitHub = git.review
GitHub Actions = cicd.runner
boto3 = cloud.inventory + cloud.action_execute_approved
```

---

## 18. Updated v0.9.8 `kiste.yaml`

```yaml
apiVersion: kiste.dev/v0.9.8
kind: Workspace

metadata:
  name: ai-product-platform
  owner: group:default/platform

spec:
  manager: {}

  repos: []

  intent:
    declared:
      - kubernetes-lize-project

  targets:
    default: dev
    environments:
      dev:
        runtime: kubernetes
        region: ap-southeast-1

  requires:
    capabilities:
      - workspace.read
      - repo.discover
      - git.state_read
      - validate.workspace
      - validate.capability
      - validate.policy
      - inspect.hardware
      - inspect.security
      - runtime.kubernetes_lize
      - runtime.manifest_generate
      - runtime.manifest_validation
      - runtime.health
      - runtime.drift
      - secrets.refs

  units:
    requires:
      - module: github.com/KisteBox/kiste-unit-kubernetes
        version: v0.1.0

  unit_policy:
    allowed_hosts:
      - github.com
      - internal.git.example.com
    require_lock_file: true
    require_checksum: true
    allow_unpinned_versions: false
    review_required_for_new_unit: true
    review_required_for_version_change: true

  git_update:
    enabled: true
    require_approved_plan: true

  policy:
    deploy:
      require_review: true
      require_approved_plan: true
```

---

## 19. Output Layout Additions

```text
.kiste/
  units/
    required-units.json
    resolved-units.json
    unit-resolution-report.json
    unit-trust-report.json
    unit-graph.json

  sdk/
    sdk-capability-report.json
    tool-weaving-report.json
    adapter-contract-report.json

  inspect/
    hardware-report.json
    security-report.json
    ai-capability-report.json

  kubernetes/
    manifest-report.json
    dry-run-plan.json
    runtime-plan.json
    runtime-monitor-plan.json

  validate/
    validation-report.json
    validation-capability-report.json
    blockers.json
    warnings.json

  review/
    review-report.json
    review-capability-report.json
    deployment-readiness-report.json

  selftest/
    deployment-self-test-plan.json
    self-test-report.json

  monitor/
    kubernetes-health.json
    runtime-drift.json
    ai-workload-drift.json
```

---

## 20. v0.9.8 Acceptance Criteria

v0.9.8 is accepted only if:

```text
1. KisteUnit rejects mode: plugin.
2. KisteUnit rejects type: plugin.
3. Workspace supports spec.units.requires.
4. Workspace supports spec.unit_policy.
5. Kiste can resolve unit module path without executing it.
6. Kiste can write kiste.unit.lock.
7. SDK exposes KisteUnit loading and validation.
8. SDK exposes capability declaration helpers.
9. SDK exposes ToolAdapter interface.
10. SDK can normalize adapter output into Kiste reports.
11. Kiste can read a project and propose Kubernetes manifests.
12. Kubernetes-lize produces PatchSet, not direct mutation.
13. Inspect includes hardware analysis report.
14. Inspect includes security analysis report.
15. Inspect blocks plaintext secret risk.
16. Inspect warns about missing resource limits.
17. validate.* capabilities are recognized.
18. review.* capabilities are recognized.
19. Non-mutating validation can run directly.
20. Mutating validation becomes SelfTestPlan.
21. AI capability taxonomy is recognized.
22. Minimum Kiste capability kernel works without external providers.
23. Existing tools can be represented as Kiste capability providers without adding top-level commands.
```

---

## 21. Non-Goals for v0.9.8

```text
central plugin marketplace
plugin mode
full GitHub Easy Mode
GitHub Actions execution lane
boto3 execution lane
Pulumi integration
Ansible integration
Terraform/OpenTofu integration
Argo CD expansion
Hugging Face runtime integration
Kaggle integration
MLflow/Kubeflow/Ray integration
vector database integration
full AI agent runtime
autonomous self-modification
```

---

## 22. Final Rule

```text
v0.9.8 expands Kiste from foundation into an SDK-driven platform model.

It removes plugin mode, makes KisteUnits decentralized, expands the SDK, Kubernetes-lizes projects, adds hardware/security/AI inspection, defines validation/review/self-test capabilities, and introduces the tool-weaving model.

Kiste core remains small through the Minimum Operating Capability Kernel.

Everything else grows through capabilities.
```
