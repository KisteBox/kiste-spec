# Kiste v0.9.7 — Platform Engineering Foundation, Standard Workspace Schema, Capability Model, Decentralized Workspace Manager, and Four-Way Architecture Split

Status: Canonical engineering specification  
Target implementation: `KisteBox/kiste-py`  
Spec repository: `KisteBox/kiste_spec`  
Release: `0.9.7`

---

## 1. Release Theme

Kiste v0.9.7 is a platform engineering foundation update.

It does not add new external tool integrations.

It standardizes Kiste itself before expanding adapters, tools, plugins, and third-party units.

```text
v0.9.7 = standardize Kiste before adding tools
```

The release focuses on:

```text
standard kiste.yaml
capability model
provider contract
Git compatibility contract
policy-gated control plane
decentralized WorkspaceManager contract
four-way architecture split:
  CLI
  Python SDK
  Core Engine
  Self-Improvement / KisteUnit Development Workspace
```

One-sentence definition:

```text
Kiste v0.9.7 standardizes the workspace schema, capability model, provider contract, decentralized workspace management, and four-way architecture split so Kiste can become a stable platform for building and improving first-party and third-party KisteUnits.
```

---

## 2. Non-Goal: No New Tool Integration

v0.9.7 must not add new real tool adapters.

No new implementation for:

```text
Kubernetes
Pulumi
Ansible
boto3
Argo CD
Helm
Kustomize
Terraform
OpenTofu
Hugging Face
Kaggle
GitHub Actions
GitLab CI
```

These tools may appear only as examples of future providers.

v0.9.7 is about the contract that future providers must follow.

```text
No new tools.
No direct wrappers.
No tool-specific commands.
No provider execution.
No raw shell orchestration.
```

---

## 3. Core Philosophy

Kiste should not be centered on tools.

Kiste should be centered on:

```text
workspace
capability
policy
plan
Git
monitor state
developer-extensible KisteUnit model
```

The core model:

```text
KisteUnit requires capabilities.
Provider declares capabilities.
Kiste resolves required capability -> approved provider.
Policy controls what the provider may read, plan, write, deploy, or monitor.
Git controls how file changes are delivered.
The six-step lifecycle controls execution.
```

A tool may provide many capabilities.

A capability may be provided by many tools.

Kiste chooses the safest allowed provider based on:

```text
workspace intent
target
policy
lifecycle stage
Git compatibility
manager trust
review/approval state
```

---

## 4. No New Top-Level Commands

v0.9.7 must not add user-facing commands such as:

```bash
kiste pulumi
kiste ansible
kiste kubernetes
kiste capability
kiste provider
kiste manager
kiste self
kiste tool
```

The canonical lifecycle remains:

```bash
kiste read
kiste inspect
kiste plan
kiste review
kiste deploy
kiste monitor
```

Everything else must fit under the lifecycle.

Subsystem commands may exist later, but v0.9.7 should not expand the CLI surface.

---

## 5. Standard `kiste.yaml` Shape

`kiste.yaml` becomes a stable API object.

Recommended top-level shape:

```yaml
apiVersion: kiste.dev/v0.9.7
kind: Workspace

metadata:
  name: ai-product-platform
  owner: group:default/platform
  description: Full-stack AI application workspace
  labels: {}
  annotations: {}

spec:
  manager: {}
  repos: []
  intent: {}
  targets: {}
  requires: {}
  providers: []
  resolution: {}
  git_update: {}
  policy: {}

status:
  observed: {}
  last_read: null
  last_inspect: null
  last_plan: null
  last_monitor: null
```

Core rule:

```text
metadata = identity
spec = desired state
policy = allowed boundary
status = observed state
plan = proposed change
review = approval record
deploy = approved mutation
monitor = feedback into next cycle
```

---

## 6. Example Standard Workspace

```yaml
apiVersion: kiste.dev/v0.9.7
kind: Workspace

metadata:
  name: ai-product-platform
  owner: group:default/platform
  description: Full-stack AI application workspace

spec:
  manager:
    mode: decentralized
    current:
      name: local-self-hosted
      type: self-hosted
      endpoint: http://localhost:8797
      trust: local
      authority: workspace-owner

    policy:
      allow_manager_switch: approved-only
      allow_remote_manager: approved-only
      allow_self_hosted_manager: true
      require_signed_manager_identity: true
      require_workspace_access_policy: true

  repos:
    - name: frontend
      path: ./frontend
      role: frontend

    - name: backend
      path: ./backend
      role: service

    - name: model
      path: ./model
      role: model-service

    - name: gitops
      path: ./gitops
      role: deployment-state

  intent:
    declared:
      - full-stack-ai-app

  targets:
    default: dev
    environments:
      dev:
        provider: aws
        region: ap-southeast-1
        runtime: kubernetes

  requires:
    capabilities:
      - cloud.inventory
      - cloud.provision
      - container.registry
      - runtime.target
      - runtime.health
      - git.update
      - secrets.refs
      - cicd.pipeline
      - observability.metrics

  providers: []

  resolution: {}

  git_update:
    enabled: true
    mode: pull_request
    require_approved_plan: true
    no_direct_main_push: true
    no_force_push: true

  policy:
    region:
      default: ap-southeast-1
      allowed:
        - ap-southeast-1
      allow_cross_region_resources: false

    cloud:
      account_boundary: dev-account
      allow_cross_account_resources: false

    secrets:
      allow_secret_values: false
      allow_secret_refs: true

    deploy:
      require_review: true
      require_approved_plan: true
```

---

## 7. Capability Model

A capability is a stable named ability that Kiste can read, inspect, plan, review, deploy, or monitor.

Capabilities should not encode tool names.

Good:

```text
cloud.provision
cloud.inventory
machine.configure
runtime.target
runtime.health
git.update
secrets.refs
observability.metrics
cicd.pipeline
```

Bad:

```text
pulumi.provision
ansible.configure
boto3.ecs.create
kubectl.apply
```

Tool-specific behavior belongs in providers and adapters later.

Core capability groups:

```text
workspace.*
repo.*
git.*
cloud.*
runtime.*
secrets.*
cicd.*
observability.*
machine.*
configuration.*
policy.*
cost.*
release.*
unit.*
sdk.*
```

Example capability list:

```text
workspace.graph
workspace.index
workspace.access
repo.read
repo.patch
git.update
git.review
git.monitor
cloud.inventory
cloud.provision
cloud.preview
cloud.resource_action
cloud.drift
container.registry
runtime.target
runtime.dry_run
runtime.manifest_validation
runtime.health
runtime.drift
secrets.refs
secrets.backend
cicd.pipeline
observability.metrics
observability.logs
machine.configure
configuration.drift
policy.validate
cost.estimate
release.tag
unit.scaffold
unit.validate
unit.package
unit.publish
sdk.test
```

---

## 8. Provider Contract

A provider declares capabilities.

In v0.9.7, providers are contracts only.

They are not implemented tool adapters.

Provider contract shape:

```yaml
providers:
  - name: future-provider
    type: provider-contract
    adapter: null

    provides:
      capabilities:
        - name: cloud.provision
          stages:
            read: true
            inspect: true
            plan: true
            deploy: approved-only
            monitor: true

    git:
      mode: hybrid
      requires_git_update_plan: true
      requires_approved_action_plan: true

    policy:
      approved_plan_required: true
      secret_values_allowed: false
```

Provider contract rule:

```text
A provider cannot be selected unless:
  it declares the required capability,
  it supports the lifecycle stage,
  it satisfies policy,
  it declares Git or ActionPlan compatibility,
  it is allowed by WorkspaceManager policy.
```

---

## 9. Git Compatibility Modes

Every future provider must declare how it works with Kiste Git.

Standard modes:

```text
render-only
  Provider only renders files for Git.

render-and-apply
  Provider renders files and may apply them after approval.

render-and-execute
  Provider renders files and executes after approval.

hybrid
  Provider may render files and execute API operations after approval.

action-plan
  Provider mainly calls APIs and must produce an approved ActionPlan.

observe-only
  Provider only reads/monitors and never mutates.
```

Hard rule:

```text
Any provider that changes files must produce Git-compatible PatchSets.
Any provider that mutates cloud/runtime/external systems must produce approved ActionPlans or ToolExecutionPlans.
```

---

## 10. Capability Resolution

Kiste resolves required capabilities to providers.

In v0.9.7, this can be implemented using provider declarations and fake/demo provider contracts only.

Resolution algorithm:

```text
1. Read required capabilities from spec.requires.capabilities.
2. Read provider declarations from spec.providers.
3. Filter providers by capability support.
4. Filter by lifecycle stage.
5. Filter by policy.
6. Filter by target and region.
7. Filter by Git compatibility.
8. Filter by WorkspaceManager trust and authority.
9. Select preferred provider if configured.
10. Record fallback and denied providers.
11. Write resolution report.
```

Example resolution block:

```yaml
resolution:
  cloud.provision:
    preferred:
      - future-pulumi-provider
    fallback:
      - future-cloud-sdk-provider
    denied:
      - raw-shell

  cloud.inventory:
    preferred:
      - future-cloud-sdk-provider

  machine.configure:
    preferred:
      - future-configuration-provider

  runtime.target:
    preferred:
      - future-kubernetes-provider
```

In v0.9.7, these are only contracts and examples.

---

## 11. Decentralized Workspace Manager

Kiste must support decentralized workspace managers.

A workspace manager coordinates workspace state, but it must not bypass Kiste lifecycle or policy.

Manager types:

```text
local
self-hosted
team-hosted
enterprise-hosted
future-cloud-hosted
```

Core rule:

```text
A Kiste workspace must not depend on one central Kiste server.
```

The workspace declares its manager.

The manager can be replaced only through approved Git change.

---

## 12. WorkspaceManager Contract

Add a standard object:

```yaml
apiVersion: kiste.dev/v0.9.7
kind: WorkspaceManager

metadata:
  name: local-self-hosted

spec:
  type: self-hosted
  authority: workspace-owner

  manages:
    workspaces:
      - ai-product-platform

  capabilities:
    - workspace.index
    - workspace.graph
    - policy.validate
    - plan.store
    - review.record
    - monitor.collect
    - unit.registry
    - unit.validate

  policy:
    require_signed_plans: true
    require_approved_deploy: true
    no_secret_values: true
    no_direct_git_mutation: true
```

Manager responsibility:

```text
workspace registry
workspace graph
KisteUnit registry
capability registry
policy enforcement
plan storage
review records
monitor status
multi-repo coordination
self-improvement plans
unit development workspace coordination
```

Manager non-responsibility:

```text
bypass review
bypass GitUpdatePlan
bypass WorkspaceAccessPolicy
read secret values
mutate repos directly
mutate cloud/runtime directly
```

---

## 13. Self-Hosted Manager

Self-hosted manager is first-class.

Example:

```yaml
spec:
  manager:
    current:
      name: kiste-self-hosted
      type: self-hosted
      endpoint: https://kiste.internal.example.com
      authority: workspace-owner

    policy:
      data_residency:
        required: true
      allowed_regions:
        - ap-southeast-1
      external_control_plane_allowed: false
```

Meaning:

```text
The workspace is managed by a self-hosted Kiste manager.
Data/control state stays inside the user boundary.
External Kiste cloud is not required.
```

---

## 14. Policy-Gated Control Plane Algorithm

v0.9.7 introduces the stable control-plane algorithm shape.

Name:

```text
Policy-Gated Capability Reconciliation
```

Algorithm:

```text
1. Read kiste.yaml.
2. Validate schema.
3. Build WorkspaceGraph.
4. Build ManagerGraph.
5. Build CapabilityGraph.
6. Build ProviderGraph.
7. Build PolicyGraph.
8. Read observed repo/cloud/runtime state.
9. Normalize observed state into Kiste facts.
10. Compare desired state against observed state.
11. Produce DriftGraph.
12. Convert drift into CandidateActions.
13. Resolve providers for required capabilities.
14. Filter actions through PolicyGate.
15. Convert file-changing actions into GitUpdatePlan.
16. Convert API-changing actions into ToolExecutionPlan or ActionPlan.
17. Require review.
18. Execute only approved plans.
19. Monitor and feed results back into status.
```

Plain rule:

```text
desired != observed does not mean mutate.
desired != observed means produce a policy-checked plan.
```

---

## 15. Action Graph

Every planned change becomes an action node.

```json
{
  "id": "action-004",
  "type": "provider.execute",
  "provider": "future-provider",
  "capability": "cloud.provision",
  "stage": "deploy",
  "requires": [
    "action-001",
    "action-002"
  ],
  "policy": {
    "approved_plan_required": true,
    "region": "ap-southeast-1"
  },
  "risk": "medium",
  "allowed_now": false,
  "allowed_after_review": true
}
```

The action graph allows Kiste to:

```text
sort dependencies
detect unsafe order
separate file changes from API actions
block unapproved mutation
generate reviewable plans
monitor drift after execution
```

---

# 16. Strong Four-Way Architecture Split

v0.9.7 must establish a strong internal split:

```text
1. CLI Layer
2. Python SDK Layer
3. Core Engine Layer
4. Self-Improvement / KisteUnit Development Workspace
```

This is one of the most important architectural decisions in the release.

The rule:

```text
Kiste core must not depend on CLI.
Kiste core must not depend on self-improvement.
Self-improvement must be a KisteUnit/workspace and SDK surface, not special core logic.
CLI must be a thin shell over Python SDK.
Python SDK must be a stable programmatic facade over core.
Third-party KisteUnit development must use the same SDK/workspace path as self-improvement.
```

---

## 17. Layer 1 — CLI Layer

Package area:

```text
src/kiste_cli/
```

Responsibility:

```text
parse command-line arguments
load current directory
call Python SDK
print human-readable output
write exit codes
format errors
```

Allowed commands:

```bash
kiste read
kiste inspect
kiste plan
kiste review
kiste deploy
kiste monitor
```

CLI must not contain:

```text
control-plane algorithms
provider resolution logic
policy engine
Git update engine
self-improvement logic
tool-specific execution logic
business logic
third-party unit scaffolding logic
```

CLI dependency direction:

```text
kiste_cli -> kiste Python SDK
```

The CLI is replaceable.

A future web UI, API server, self-hosted manager, or third-party dev tool should be able to use the same Python SDK without depending on CLI internals.

---

## 18. Layer 2 — Python SDK Layer

Package area:

```text
src/kiste/
```

Responsibility:

```text
stable public SDK
programmatic lifecycle functions
typed request/response objects
compatibility facade
simple import path for users
developer-facing KisteUnit APIs
```

Public lifecycle API shape:

```python
import kiste

read_report = kiste.read(".")
inspection = kiste.inspect(".")
plan = kiste.plan(".", intent="full-stack-ai-app")
review = kiste.review(".kiste/plans/latest.json", decision="approve")
deploy = kiste.deploy(".kiste/reviews/latest.approved.json")
monitor = kiste.monitor(".", once=True)
```

Public KisteUnit development API shape:

```python
import kiste

unit = kiste.unit.load("./my-kiste-unit")
result = kiste.unit.validate(unit)
package = kiste.unit.package(unit)
plan = kiste.unit.plan_against_workspace(
    unit=unit,
    workspace=".",
)
```

The Python SDK should expose stable objects:

```text
Workspace
WorkspaceManager
KisteUnit
Capability
ProviderContract
ActionGraph
PolicyReport
ReadReport
InspectionReport
PlanReport
ReviewReport
DeployReport
MonitorReport
UnitValidationReport
UnitPackageReport
```

Python SDK must not contain:

```text
CLI parsing
terminal formatting
hardcoded self-improvement workflow
tool-specific shortcuts
unsafe mutation shortcuts
provider execution logic
```

Python SDK dependency direction:

```text
kiste -> kiste_core
```

The Python SDK is the stable contract for:

```text
CLI
self-hosted manager
tests
future web UI
future automation API
self-improvement workspace
third-party KisteUnit developers
```

---

## 19. Layer 3 — Core Engine Layer

Package area:

```text
src/kiste_core/
```

Responsibility:

```text
schema parsing and validation
workspace graph
manager graph
capability graph
provider graph
policy graph
observed state normalization
desired vs observed diff
action graph generation
policy gate
plan generation
review validation
deploy validation
monitor state model
output writers
KisteUnit contract validation
KisteUnit package validation
```

Recommended module layout:

```text
src/kiste_core/
  schema/
    workspace.py
    manager.py
    capability.py
    provider.py
    policy.py
    unit.py

  workspace/
    models.py
    graph.py
    loader.py
    status.py

  manager/
    models.py
    registry.py
    trust.py
    policy.py

  capabilities/
    models.py
    registry.py
    resolver.py
    graph.py
    report.py

  providers/
    models.py
    registry.py
    contract.py
    policy.py
    git_modes.py

  units/
    models.py
    loader.py
    validator.py
    packager.py
    registry.py
    graph.py

  control_plane/
    desired.py
    observed.py
    normalize.py
    diff.py
    action_graph.py
    planner.py
    policy_gate.py

  lifecycle/
    read.py
    inspect.py
    plan.py
    review.py
    deploy.py
    monitor.py

  output/
    writer.py
    paths.py
    schemas.py
```

Core must not import:

```text
kiste_cli
kiste_self_improvement
third-party unit code directly
provider-specific adapters that are not contract-only
external tool SDKs by default
```

Core may define interfaces and schemas.

Core must stay pure.

---

## 20. Layer 4 — Self-Improvement / KisteUnit Development Workspace

Self-improvement must be separated from Kiste core.

But it should be more than “Kiste improves itself.”

In v0.9.7, self-improvement becomes the first developer workspace and SDK pattern for building, testing, validating, packaging, and publishing KisteUnits.

Better name:

```text
Kiste Development Workspace
```

or:

```text
KisteUnit SDK Workspace
```

The self-improvement layer has two responsibilities:

```text
1. Improve Kiste itself through normal KisteUnit workflow.
2. Let developers build third-party KisteUnits using the same workflow.
```

This layer should become the place where developers can:

```text
scaffold a KisteUnit
declare capabilities
declare provider contracts
validate policy compatibility
test against sample workspaces
generate documentation
package the unit
publish to a unit registry later
```

This should not live in core.

---

## 21. Self-Improvement as SDK / Workspace

Recommended package/project direction:

```text
src/kiste_unit_sdk/
```

or later as a separate project:

```text
kiste-unit-sdk/
kiste-unit-self-improvement/
```

Responsibility:

```text
unit scaffolding
unit validation helpers
unit test harness
sample workspace runner
spec/code drift detection
Kiste self-improvement plans
third-party unit packaging
third-party unit documentation generation
unit registry metadata generation
```

This layer may provide developer APIs such as:

```python
from kiste_unit_sdk import UnitBuilder, UnitTestHarness

unit = (
    UnitBuilder("aws-cost-advisor")
    .requires("cloud.inventory")
    .provides("cost.estimate")
    .with_policy("read_only_by_default", True)
    .build()
)

harness = UnitTestHarness(unit)
harness.run_against("./examples/workspaces/aws-dev")
```

It may also support Kiste self-improvement:

```python
from kiste_unit_sdk import SelfImprovementWorkspace

workspace = SelfImprovementWorkspace(
    spec_repo="./kiste_spec",
    code_repo="./kiste-py",
)

plan = workspace.plan_spec_code_alignment()
```

The same SDK path is used for:

```text
Kiste improving Kiste
developers building third-party KisteUnits
teams building internal KisteUnits
vendors building commercial KisteUnits
```

---

## 22. Self-Improvement Must Not Be Special Core Logic

Self-improvement must not:

```text
mutate Kiste core directly
bypass GitUpdatePlan
bypass review
auto-merge by default
be required for normal Kiste operation
be imported by kiste_core
be hidden inside CLI commands
```

Self-improvement dependency direction:

```text
kiste_unit_sdk -> kiste Python SDK
kiste_unit_sdk -> kiste_core contracts if needed
```

Forbidden:

```text
kiste_core -> kiste_unit_sdk
kiste_core -> self-improvement
kiste_cli -> self-improvement as a special case
```

Self-improvement is a user of Kiste, not the owner of Kiste.

---

## 23. Four-Way Dependency Rule

Allowed dependency graph:

```text
kiste_cli
  -> kiste
      -> kiste_core

kiste_unit_sdk / self-improvement workspace
  -> kiste
      -> kiste_core
```

Forbidden dependency graph:

```text
kiste_core -> kiste_cli
kiste_core -> kiste_unit_sdk
kiste_core -> self-improvement
kiste -> kiste_cli
kiste_cli -> kiste_core internals
kiste_unit_sdk -> kiste_cli
self-improvement -> kiste_cli
```

Best rule:

```text
Core is pure.
Python SDK is stable.
CLI is thin.
Self-improvement is external.
KisteUnit development uses the SDK/workspace layer.
```

---

## 24. Self-Improvement as KisteUnit

Self-improvement should be modeled like this:

```yaml
apiVersion: kiste.dev/v0.9.7
kind: KisteUnit

metadata:
  name: self-improve-kiste
  owner: group:default/kiste-core

spec:
  intent:
    declared:
      - self-improve-kiste

  requires:
    capabilities:
      - repo.read
      - git.update
      - policy.validate
      - release.plan
      - cicd.pipeline
      - unit.validate
      - sdk.test

  provides:
    capabilities:
      - spec.drift.detect
      - code.drift.detect
      - unit.scaffold
      - unit.validate
      - unit.package
      - release.plan

  policy:
    self_modification:
      auto_apply: false
      auto_merge: false
      require_review: true
      require_approved_plan: true

  access:
    read:
      repos: full
      secrets: refs-only

    patch:
      allowed: true
      paths:
        allow:
          - src/**
          - tests/**
          - docs/**
          - Phase*/**
        deny:
          - secrets/**
          - .env
          - "*.pem"

    deploy:
      allowed: false
```

In v0.9.7, this is a contract.

Actual autonomous execution remains later.

---

## 25. Third-Party KisteUnit Development Contract

v0.9.7 should define how developers will build third-party KisteUnits.

Third-party KisteUnit structure:

```text
my-kiste-unit/
  kiste.unit.yaml
  README.md
  examples/
    workspace/
      kiste.yaml
  tests/
    test_unit_contract.py
  src/
    my_kiste_unit/
      __init__.py
```

Example `kiste.unit.yaml`:

```yaml
apiVersion: kiste.dev/v0.9.7
kind: KisteUnit

metadata:
  name: aws-cost-advisor
  owner: user:example/dev

spec:
  requires:
    capabilities:
      - cloud.inventory
      - cost.estimate

  provides:
    capabilities:
      - cost.report
      - cost.recommendation

  policy:
    read_only_by_default: true
    cloud_mutation_allowed: false
    secret_values_allowed: false

  git:
    mode: observe-only

  tests:
    required:
      - schema.valid
      - policy.valid
      - no_secret_values
      - no_mutation_without_plan
```

Third-party unit rules:

```text
Must declare capabilities.
Must declare policy.
Must declare Git or ActionPlan compatibility.
Must include tests.
Must not mutate by default.
Must not read secret values by default.
Must be validated before use.
```

---

## 26. KisteUnit SDK Responsibilities

The KisteUnit SDK should eventually provide:

```text
unit scaffold
unit schema validation
unit capability validation
unit policy validation
unit test harness
sample workspace runner
mock WorkspaceContext
mock WorkspaceAccessPolicy
mock GitUpdatePlan
mock ActionPlan
documentation generator
package builder
registry metadata generator
```

But in v0.9.7, only the contract is required.

The implementation can be minimal.

---

## 27. Lifecycle Integration

### Read

Reads:

```text
standard kiste.yaml
workspace manager declaration
repos
required capabilities
provider declarations
Git state
policy
status
KisteUnit declarations
SDK/unit metadata if present
```

Outputs:

```text
WorkspaceGraph
ManagerGraph
CapabilityGraph
ProviderGraph
PolicyGraph
UnitGraph
```

### Inspect

Validates:

```text
schema validity
manager contract
required capabilities
provider declarations
Git compatibility modes
policy boundaries
four-way architecture boundaries
self-improvement separation
third-party unit contract
```

### Plan

Produces:

```text
CapabilityResolutionReport
ActionGraph
GitUpdatePlan if needed
ToolExecutionPlan contract if needed
ActionPlan contract if needed
RuntimePlan contract if needed
UnitValidationPlan
MonitorPlan
RollbackPlan
```

### Review

Shows:

```text
selected capabilities
selected providers
selected units
manager authority
denied actions
Git changes
API actions
runtime changes
unit risks
policy violations
rollback
```

### Deploy

Executes only approved plans.

In v0.9.7, deploy may remain report-only for provider execution, but it must validate that no provider/action bypasses approval.

### Monitor

Watches:

```text
workspace drift
manager drift
capability drift
provider drift
unit drift
Git drift
policy drift
architecture boundary drift
```

---

## 28. Output Layout

```text
.kiste/
  workspace/
    workspace.json
    workspace-graph.json
    status.json

  manager/
    manager.json
    manager-graph.json
    manager-policy-report.json

  capabilities/
    required.json
    provided.json
    resolution-report.json
    denied-providers.json

  providers/
    provider-registry.json
    provider-state.json
    git-compatibility.json

  units/
    unit-graph.json
    unit-validation-report.json
    unit-package-report.json
    third-party-unit-report.json

  control-plane/
    workspace-graph.json
    manager-graph.json
    capability-graph.json
    provider-graph.json
    unit-graph.json
    policy-graph.json
    drift-graph.json
    action-graph.json

  architecture/
    layer-report.json
    dependency-boundary-report.json
    self-improvement-boundary-report.json
    sdk-boundary-report.json

  plans/
    git-update-plan.json
    tool-execution-plan.json
    action-plan.json
    runtime-plan.json
    unit-validation-plan.json

  inspect/
    schema-report.json
    capability-report.json
    provider-report.json
    manager-report.json
    unit-report.json
    architecture-report.json

  monitor/
    workspace-drift.json
    manager-drift.json
    capability-drift.json
    provider-health.json
    unit-drift.json
    architecture-drift.json
```

---

## 29. Architecture Boundary Checks

v0.9.7 should add architecture checks.

Required checks:

```text
architecture.cli_is_thin
architecture.python_sdk_is_stable_facade
architecture.core_has_no_cli_import
architecture.core_has_no_self_improvement_import
architecture.core_has_no_unit_sdk_import
architecture.self_improvement_not_required
architecture.unit_sdk_not_required_for_core
architecture.no_tool_adapter_in_core_default_path
architecture.no_new_tool_command
architecture.lifecycle_commands_only
```

These checks prevent Kiste from becoming tangled before v1.0.

---

## 30. Safety Rules

Required safety rules:

```text
No tool integration in v0.9.7.
No provider execution without policy.
No file mutation outside GitUpdatePlan.
No cloud mutation outside approved ActionPlan.
No runtime mutation without approved deploy.
No secret values in provider output.
No raw shell as a default provider.
No centralized manager requirement.
No manager switch without approved Git change.
No self-improvement import inside core.
No self-improvement auto-apply.
No third-party unit execution without validation.
No third-party unit mutation by default.
No CLI business logic.
No Python SDK dependency on CLI.
No core dependency on CLI, self-improvement, or unit SDK.
```

---

## 31. Acceptance Tests

v0.9.7 is accepted only if:

```text
1. kiste --version returns 0.9.7.
2. kiste.yaml supports apiVersion: kiste.dev/v0.9.7.
3. kiste.yaml supports kind: Workspace.
4. kiste inspect validates standardized schema shape.
5. kiste read extracts required capabilities.
6. kiste inspect validates capability names.
7. kiste inspect validates provider declarations as contracts.
8. kiste inspect rejects providers without Git compatibility mode.
9. kiste plan can build CapabilityGraph.
10. kiste plan can build ProviderGraph.
11. kiste plan can build PolicyGraph.
12. kiste plan can build ActionGraph.
13. kiste plan can build UnitGraph.
14. kiste plan rejects unresolved capabilities.
15. kiste plan records denied providers.
16. kiste read supports decentralized manager declaration.
17. kiste inspect validates WorkspaceManager contract.
18. kiste inspect validates self-hosted manager policy.
19. kiste inspect rejects unauthorized manager switch.
20. No new top-level commands are added.
21. No real Kubernetes/Pulumi/Ansible/boto3 adapter is required.
22. Core does not import CLI.
23. Core does not import self-improvement.
24. Core does not import KisteUnit SDK.
25. CLI calls Python SDK rather than core internals.
26. Self-improvement is modeled as external KisteUnit/workspace.
27. KisteUnit SDK/workspace contract exists for third-party units.
28. Third-party KisteUnit schema can be validated.
29. Architecture boundary report is generated.
30. Unit validation report is generated.
```

---

## 32. Non-Goals

v0.9.7 does not include:

```text
new top-level tool commands
new real external tool adapters
Kubernetes implementation
Pulumi implementation
Ansible implementation
boto3 implementation
Argo CD expansion
Hugging Face integration
Kaggle integration
full plugin marketplace
full hosted control plane
autonomous self-modification
auto-merge
raw shell orchestration
provider execution without approved plan
fully implemented third-party marketplace
```

---

## 33. Migration Notes

Existing older shape:

```yaml
kiste: "0.9.x"
schema: "kiste.workspace/v0.9.x"
kind: Workspace
```

should migrate toward:

```yaml
apiVersion: kiste.dev/v0.9.7
kind: Workspace
metadata: {}
spec: {}
status: {}
```

Temporary compatibility may be allowed, but v1.0 should prefer the API-object shape.

Old subsystem commands may remain as compatibility helpers but must not become the main product journey.

---

## 34. Final Rule

```text
v0.9.7 standardizes Kiste before expanding tools.

kiste.yaml becomes the stable workspace contract.
Capabilities become the stable abstraction.
Providers become contracts before implementations.
Workspace management becomes decentralized by default.
Self-hosted managers are first-class.
The control plane becomes policy-gated capability reconciliation.

The architecture splits strongly into:
  CLI
  Python SDK
  Core Engine
  Self-Improvement / KisteUnit Development Workspace

Kiste core stays pure.
Python SDK stays stable.
CLI stays thin.
Self-improvement stays external.
The KisteUnit SDK/workspace becomes the path for developers to improve Kiste and build third-party KisteUnits.

Tools remain replaceable future providers.
Kiste remains the lifecycle controller.
```
