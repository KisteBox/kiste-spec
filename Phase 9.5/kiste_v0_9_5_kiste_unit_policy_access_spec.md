# Kiste v0.9.5 — KisteUnit and Policy-Governed Workspace Access Spec

Status: Canonical engineering specification  
Source of truth: `KisteBox/kiste_spec`  
Target implementation: `KisteBox/kiste-py`  
Release: `0.9.5`

---

## 1. Release Theme

Kiste v0.9.5 introduces the **KisteUnit** model and the **WorkspaceAccessPolicy** model.

A KisteUnit is a Kiste-shaped reusable workspace that can behave as:

- an intent
- a plugin
- a helper library
- a cloud capability provider
- a self-improvement unit
- a full-stack deployment unit

The distinction between intent and plugin is contextual, not structural.

```text
A unit used as a top-level goal behaves like an intent.
A unit used by another unit behaves like a plugin.
A unit can use other units.
A workspace can pass controlled context into a unit.
Policy decides what the unit can read, inspect, plan, write, deploy, or monitor.
```

---

## 2. Core Philosophy

```text
Everything Kiste manages should be Kiste-shaped.
```

That means:

```text
Application workspace = Kiste workspace
Intent = KisteUnit
Plugin = KisteUnit
Cloud helper = KisteUnit
Self-hosting = KisteUnit
Kiste itself = Kiste workspace
```

The key rule:

```text
KisteUnit is the universal unit of reusable Kiste logic.
```

But a KisteUnit must not receive unlimited access.

```text
KisteUnit is powerful only through granted context.
```

---

## 3. No New Top-Level Commands

v0.9.5 must not add new top-level commands such as:

```bash
kiste intent
kiste plugin
kiste registry
kiste tool
```

The canonical user-facing lifecycle remains:

```bash
kiste read
kiste inspect
kiste plan
kiste review
kiste deploy
kiste monitor
```

KisteUnit is internal architecture behind these six commands.

---

## 4. KisteUnit Definition

A KisteUnit is a Kiste workspace that declares:

```text
name
version
capabilities provided
capabilities required
other units used
external tools used
entrypoints
lifecycle support
safety rules
workspace access policy
region/account/data residency restrictions
inputs
outputs
tests
doctor groups
```

A KisteUnit should not rely on raw `runnable` or `composable` flags as the main truth.

Instead, Kiste derives behavior from:

```text
entrypoints
capabilities
lifecycle support
accepted context types
policy
```

A unit is considered runnable when it declares a top-level entrypoint and implements enough lifecycle stages to produce a plan.

A unit is considered composable when it declares a dependency entrypoint and provides capabilities that other units can require.

---

## 5. Unified KisteUnit YAML Schema

```yaml
kiste: "0.9.5"
schema: "kiste.unit/v0.9.5"
kind: KisteUnit

unit:
  name: full-stack-ai-app
  type: application-intent
  owner: group:default/kiste-core

entrypoints:
  top_level:
    allowed: true
    intent_name: full-stack-ai-app

  dependency:
    allowed: true
    accepts:
      - WorkspaceContext
      - RepoContext
      - TargetContext

provides:
  capabilities:
    - full-stack-deployment
    - workspace-catalog
    - gitops-delivery

requires:
  capabilities:
    - container-build
    - secret-management
    - kubernetes-runtime
    - monitoring

uses:
  units:
    - cncf-minimal-gitops
    - pytorch-model-service
    - secure-secret-handling
    - aws-cloud

safety:
  require_review: true
  require_approved_plan: true
  read_only_by_default: true
```

---

## 6. Intent and Plugin Unification

Intent and plugin are the same kind of object in Kiste's eyes.

Both are KisteUnits.

```text
A unit used as a goal behaves like an intent.
A unit used as a dependency behaves like a plugin.
```

Example plugin-like unit:

```yaml
kiste: "0.9.5"
schema: "kiste.unit/v0.9.5"
kind: KisteUnit

unit:
  name: argocd
  type: plugin

entrypoints:
  top_level:
    allowed: false

  dependency:
    allowed: true
    accepts:
      - WorkspaceContext
      - GitOpsContext
      - RuntimeContext

provides:
  capabilities:
    - gitops-delivery
    - deployment-health
    - runtime-drift-detection

uses:
  tools:
    - argocd

implements:
  lifecycle:
    read: true
    inspect: true
    plan: true
    deploy: true
    monitor: true

safety:
  read_only_by_default: true
  require_approved_plan_for_deploy: true
```

---

## 7. Intent Can Use Other Intent

One KisteUnit can use another KisteUnit.

Therefore, one intent can compose other intents.

Example:

```text
full-stack-ai-app
  uses cncf-minimal-gitops
  uses pytorch-model-service
  uses secure-secret-handling
  uses monitor-runtime-drift
  uses aws-cloud
```

This creates a unit graph:

```text
KisteUnit -> KisteUnit -> KisteUnit
```

Kiste resolves this graph during `read`, validates it during `inspect`, and turns it into a plan during `plan`.

---

## 8. Workspace Can Produce Intent

A normal workspace can declare intent:

```yaml
intents:
  declared:
    - full-stack-ai-app
```

Kiste can also detect intent from workspace facts.

Example:

```text
frontend repo + backend repo + model repo + GitOps manifests + secrets
```

can produce:

```text
Detected intent: full-stack-ai-app
Detected intent: pytorch-model-service
Detected intent: secret-trust-required
Detected intent: gitops-delivery
Detected intent: aws-deployment-candidate
```

Rule:

```text
Read detects intent.
Inspect validates intent.
Plan proposes action.
Review approves action.
Deploy executes approved action.
Monitor observes result.
```

---

## 9. Workspace Linking

A Kiste workspace may pass repos, config, graph, runtime state, and intent context into another KisteUnit.

This makes a KisteUnit behave like a reusable workspace-level function.

Example:

```yaml
uses:
  units:
    - name: secure-secret-handling
      source: registry
      inputs:
        repos:
          - backend
          - model

    - name: pytorch-model-service
      source: registry
      inputs:
        repos:
          - model

    - name: aws-cloud
      source: registry
      inputs:
        repos:
          - frontend
          - backend
          - model
        target:
          provider: aws
          runtime: kubernetes
          environment: dev
          region: ap-southeast-1
```

Kiste passes a controlled `WorkspaceContext` into each unit.

---

## 10. WorkspaceContext Model

```json
{
  "schema": "kiste.workspace-context/v0.9.5",
  "workspace": {
    "name": "ai-product-platform",
    "path": "."
  },
  "repos": [
    {
      "name": "frontend",
      "type": "frontend",
      "path": "./frontend"
    },
    {
      "name": "backend",
      "type": "service",
      "path": "./backend"
    },
    {
      "name": "model",
      "type": "model-service",
      "path": "./model"
    }
  ],
  "target": {
    "provider": "aws",
    "runtime": "kubernetes",
    "environment": "dev",
    "region": "ap-southeast-1"
  },
  "intent": "full-stack-ai-app"
}
```

---

## 11. WorkspaceAccessPolicy

The most important v0.9.5 safety model is `WorkspaceAccessPolicy`.

A KisteUnit may receive another workspace as input, but it only gets the access granted by policy.

```text
Unit imports context, not unlimited power.
```

Access must be explicit.

---

## 12. Access Modes

Kiste must define standard access modes:

```text
none          no access
metadata      names, types, graph, declared config only
blackbox      contract only, no internals
read          can read files/facts, no modification
inspect       can produce findings
plan          can propose changes
patch         can generate file patches, not apply
write         can modify workspace files after approval
deploy        can change runtime/cloud after approved plan
monitor       can observe runtime state
admin         full control, rare
```

Default for most units:

```text
metadata + inspect + plan
```

Not write.  
Not deploy.  
Not admin.

---

## 13. Blackbox Workspace Access

A workspace can expose itself as a blackbox.

```text
Blackbox mode = KisteUnit can see contract, not internals.
```

Example:

```yaml
access:
  mode: blackbox
  exposes:
    - repo_roles
    - service_contracts
    - api_contracts
    - ports
    - dependencies
    - deployment_target
  hides:
    - source_code
    - secrets
    - private_docs
    - raw_data
```

An AWS Cloud Unit does not need to read all source code.

It may only need:

```text
frontend exists
backend exposes port 3000
model needs GPU true/false
secrets required
target environment = dev
region = ap-southeast-1
```

---

## 14. Workspace Access Policy YAML

```yaml
workspace_access:
  units:
    - name: aws-cloud
      source: registry

      access:
        mode: blackbox

        read:
          repos: metadata
          graph: true
          config: true
          runtime: false
          secrets: refs-only

        inspect:
          allowed: true

        plan:
          allowed: true
          outputs:
            - cloud-plan
            - cost-plan
            - iam-plan
            - deployment-plan

        patch:
          allowed: false

        write:
          allowed: false

        deploy:
          allowed: false

        monitor:
          allowed: true
          runtime: health-only
```

This means:

```text
AWS unit can help design the AWS deployment.
It cannot change repo files.
It cannot mutate AWS.
It cannot read secret values.
```

---

## 15. Approved Access Escalation

Access can be expanded only after review and approved plan.

```yaml
approved_access:
  unit: aws-cloud
  approved_plan: .kiste/reviews/aws-cloud.approved.json

  deploy:
    allowed: true
    cloud_mutation: true

  write:
    allowed: true
    paths:
      - .kiste/exports/**
      - gitops/**
```

No unit may escalate itself.

Only the Kiste lifecycle may grant approved access.

---

## 16. Region Policy

Region must be first-class.

```yaml
policy:
  region:
    default: ap-southeast-1
    allowed:
      - ap-southeast-1
      - ap-east-1
    denied:
      - us-east-1
      - eu-west-1
    allow_cross_region_resources: false

  data_residency:
    required: true
    allowed_regions:
      - ap-southeast-1

  cloud:
    provider: aws
    account_boundary: dev-account
    allow_cross_account_resources: false
```

Doctor check example:

```text
FAIL aws-cloud.region.violation
Reason: proposed S3 bucket region us-east-1 is not allowed.
Allowed: ap-southeast-1
```

---

## 17. Change Boundary

Kiste must separate what can be read from what can be changed.

```yaml
change_boundary:
  files:
    allow_write:
      - .kiste/exports/**
      - gitops/**
    deny_write:
      - src/**
      - secrets/**
      - kiste.yaml

  cloud:
    allow_mutation: false

  runtime:
    allow_kubernetes_apply: false

  secrets:
    allow_secret_values: false
    allow_secret_refs: true
```

This lets a unit generate plans and manifests without touching app code.

---

## 18. Workspace Access Token

Each time one workspace passes itself to another unit, Kiste should create a scoped access contract.

This is not necessarily a secret token.

It is a permissioned context object.

```json
{
  "schema": "kiste.workspace-access/v0.9.5",
  "caller": "ai-product-platform",
  "unit": "aws-cloud",
  "mode": "blackbox",
  "permissions": {
    "read_graph": true,
    "read_repo_metadata": true,
    "read_source_code": false,
    "read_secret_values": false,
    "plan_changes": true,
    "write_files": false,
    "deploy": false,
    "monitor": true
  },
  "region_policy": {
    "allowed": ["ap-southeast-1"],
    "allow_cross_region": false
  },
  "change_boundary": {
    "allow_write": [".kiste/exports/**", "gitops/**"],
    "deny_write": ["src/**", "secrets/**", "kiste.yaml"]
  }
}
```

The unit receives this scoped context, not unlimited filesystem or cloud access.

---

## 19. KisteUnit Runtime Interface

```python
class KisteUnit:
    name: str
    version: str
    capabilities: list[str]

    def read(self, context: WorkspaceContext, access: WorkspaceAccessPolicy) -> ReadReport:
        ...

    def inspect(self, context: WorkspaceContext, access: WorkspaceAccessPolicy) -> InspectionReport:
        ...

    def plan(self, context: WorkspaceContext, access: WorkspaceAccessPolicy) -> PlanReport:
        ...

    def deploy(self, approved_plan: ApprovedPlan, access: WorkspaceAccessPolicy) -> DeploymentReport:
        ...

    def monitor(self, context: WorkspaceContext, access: WorkspaceAccessPolicy) -> MonitorReport:
        ...
```

Not every unit must implement every method.

All methods must respect access policy.

---

## 20. AWS Cloud Unit Example

The AWS Cloud Unit helps a full-stack Kiste workspace deploy on AWS.

It is not the owner of the app goal.

It is the AWS mapping helper.

```text
Full-stack workspace = what to deploy
AWS Cloud Unit = how to map it safely onto AWS
```

Example unit:

```yaml
kiste: "0.9.5"
schema: "kiste.unit/v0.9.5"
kind: KisteUnit

unit:
  name: aws-cloud
  type: cloud-unit

entrypoints:
  top_level:
    allowed: false

  dependency:
    allowed: true
    accepts:
      - WorkspaceContext
      - RepoContext
      - TargetContext
      - RegionPolicy
      - CostPolicy

provides:
  capabilities:
    - aws-networking
    - aws-compute-target
    - aws-container-registry
    - aws-secret-integration
    - aws-cost-estimation
    - aws-observability
    - aws-iam-readiness

requires:
  capabilities:
    - key-management
    - approved-plan
    - cloud-credentials-check

uses:
  tools:
    - pulumi
    - aws-cli
    - kubernetes
    - argocd

safety:
  read_only_by_default: true
  require_approved_plan_for_deploy: true
  no_cloud_mutation_during_read: true
  no_cloud_mutation_during_inspect: true
  no_secret_values_in_plan: true
```

---

## 21. AWS Cloud Unit Behavior

Given a full-stack workspace:

```text
frontend
backend
model
data
secrets
GitOps
```

The AWS Cloud Unit may recommend:

```text
ECR for container images
EKS or ECS depending on runtime requirement
S3 for model/data artifacts
IAM roles for service access
KMS for encryption
Secrets Manager, SOPS, or External Secrets for secrets
CloudWatch, Prometheus, or OpenTelemetry for observability
Argo CD for Kubernetes GitOps
```

But it must return a plan, not mutate AWS directly.

---

## 22. AWS Cloud Unit Plan Output

```json
{
  "schema": "kiste.plan/v0.9.5",
  "unit": "aws-cloud",
  "status": "ready-for-review",
  "recommended_architecture": {
    "container_registry": "ecr",
    "runtime": "eks",
    "gitops": "argocd",
    "secrets": "sops-or-secrets-manager",
    "storage": "s3",
    "encryption": "kms",
    "region": "ap-southeast-1"
  },
  "risks": [
    "GPU model serving may increase cost",
    "EKS adds operational overhead",
    "IAM roles must be scoped per service"
  ],
  "policy_checks": [
    {
      "name": "region.allowed",
      "status": "pass"
    },
    {
      "name": "data_residency.allowed_region",
      "status": "pass"
    }
  ],
  "changes": [
    "create AWS infrastructure plan",
    "generate Kubernetes manifests",
    "generate Argo CD applications",
    "generate SecretRef mapping",
    "generate monitor plan"
  ],
  "requires_review": true
}
```

---

## 23. Self-Hosting as KisteUnit

Self-hosting and self-improvement must move out of special command logic.

Instead of:

```bash
kiste self improve
```

v0.9.5 should prepare:

```bash
kiste plan --intent self-improve-kiste
```

The unit:

```yaml
kiste: "0.9.5"
schema: "kiste.unit/v0.9.5"
kind: KisteUnit

unit:
  name: self-improve-kiste
  type: system-unit

entrypoints:
  top_level:
    allowed: true
    intent_name: self-improve-kiste

  dependency:
    allowed: true

provides:
  capabilities:
    - self-inspection
    - spec-code-drift-detection
    - patch-planning
    - release-readiness

safety:
  require_review: true
  require_approved_plan: true
  auto_apply: false
  auto_merge: false
  no_unreviewed_self_modification: true

workspace_access:
  access:
    mode: read
    read:
      repos: full
      graph: true
      config: true
      secrets: refs-only

    plan:
      allowed: true

    patch:
      allowed: true

    write:
      allowed: false

    deploy:
      allowed: false
```

At v1.0.0, self-hosting should split out into a separate KisteUnit project.

---

## 24. Registry Model

The registry indexes KisteUnits.

It should not expose a new top-level CLI.

```yaml
registry:
  units:
    - name: full-stack-ai-app
      kind: KisteUnit
      type: application-intent
      path: registry/units/full-stack-ai-app

    - name: aws-cloud
      kind: KisteUnit
      type: cloud-unit
      path: registry/units/aws-cloud

    - name: argocd
      kind: KisteUnit
      type: plugin
      path: registry/units/argocd

    - name: self-improve-kiste
      kind: KisteUnit
      type: system-unit
      path: registry/units/self-improve-kiste
```

---

## 25. Lifecycle Integration

### Read

`kiste read` loads:

```text
workspace facts
declared units
detected intents
unit registry metadata
unit dependency graph
workspace access policies
region/account/data residency policies
```

### Inspect

`kiste inspect` validates:

```text
unit compatibility
capability resolution
safety rules
missing dependencies
tool fit
secret strategy
region policy
account boundary
data residency
change boundary
cost risk
```

### Plan

`kiste plan` resolves:

```text
intent -> unit graph -> capabilities -> tools -> policy-checked plan
```

### Review

`kiste review` presents:

```text
selected units
selected tools
granted access
denied access
region/account boundaries
risks
cost implications
security implications
rollback strategy
```

### Deploy

`kiste deploy` executes only:

```text
approved plan
approved access
approved region/account boundary
```

### Monitor

`kiste monitor` watches:

```text
runtime health
drift
cost anomalies
security regression
unit/plugin failures
region drift
account drift
secret drift
```

---

## 26. Doctor Groups

v0.9.5 adds or formalizes:

```text
unit
intent
registry
capability
workspace-access
policy
region
account
data-residency
change-boundary
self-improvement
```

Example checks:

```text
unit.schema.valid
unit.entrypoints.valid
unit.capabilities.resolved
unit.no_cycle_without_policy

workspace_access.mode.valid
workspace_access.no_secret_value_access_by_default
workspace_access.no_write_without_approval
workspace_access.no_deploy_without_approved_plan

policy.region.allowed
policy.region.no_cross_region_resource
policy.account.boundary_valid
policy.data_residency.allowed
policy.change_boundary.valid

self_improvement.no_auto_merge
self_improvement.review_required
```

---

## 27. Output Layout

```text
.kiste/
  units/
    loaded-units.json
    unit-graph.json
    unit-decisions.json
    capability-resolution.json

  intent/
    detected-intents.json
    selected-intents.json
    resolved-intents.json

  registry/
    units.json
    capabilities.json
    tools.json

  policy/
    workspace-access.json
    region-policy.json
    account-policy.json
    data-residency-policy.json
    change-boundary.json

  plans/
    unit-plan.json
    aws-cloud-plan.json
    full-stack-ai-app-plan.json

  inspect/
    unit-compatibility-report.json
    workspace-access-report.json
    policy-report.json

  monitor/
    unit-health.json
    unit-drift.json
    policy-drift.json
```

---

## 28. Safety Rules

KisteUnit must not bypass lifecycle safety.

Required rules:

```text
No workspace mutation during read.
No cloud mutation during read or inspect.
No deploy without approved plan.
No secret values in unit output.
No self-modification without review.
No plugin execution without declared capability.
No tool usage without mapped capability.
No production placeholder deployment.
No region crossing without policy.
No account crossing without policy.
No data residency violation.
No source-code access in blackbox mode.
No write access unless policy and approved plan allow it.
```

---

## 29. Acceptance Tests

v0.9.5 is accepted only if:

```text
1. kiste --version returns 0.9.5.
2. kiste config schema supports kiste.unit/v0.9.5.
3. kiste read can load declared units from workspace YAML.
4. kiste read can detect candidate intents from workspace facts.
5. kiste inspect validates unit compatibility.
6. kiste inspect validates workspace access policy.
7. kiste plan resolves one intent into multiple units.
8. One unit can use another unit.
9. AWS Cloud Unit can receive frontend/backend/model repo context.
10. AWS Cloud Unit returns an AWS plan without mutating AWS.
11. AWS Cloud Unit respects allowed region policy.
12. AWS Cloud Unit fails if it proposes a denied region.
13. blackbox mode hides source code and secret values.
14. deploy refuses unapproved unit plans.
15. deploy refuses unapproved access escalation.
16. self-improve-kiste exists as a KisteUnit, not special command logic.
17. plugin-like units and intent-like units use the same schema.
18. no new top-level commands are required.
19. monitor can report unit-level health/drift.
20. doctor can detect unit/policy/version drift.
```

---

## 30. Final Rule

```text
In v0.9.5, Kiste unifies intent and plugin as KisteUnit.

A KisteUnit is a Kiste-shaped workspace that can be used as a goal, helper, plugin,
cloud provider unit, or self-improvement unit.

A unit can use other units.
A workspace can pass repos, config, graph, runtime state, and intent context into a unit.

But every cross-workspace interaction must be governed by WorkspaceAccessPolicy.

No unit may read secrets, mutate files, change cloud resources, deploy workloads,
or cross region/account/data-residency boundaries unless policy and an approved plan allow it.

KisteUnit may improve a plan, but it must not mutate the workspace, cloud, or Kiste itself
outside the approved six-step lifecycle.
```
